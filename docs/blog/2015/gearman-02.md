# Gearman（二）

:material-clock-time-three-outline: 2015-09-06 10:49

## Simple Demo

下面是一个简单的仅仅是输出Hello world的实例。

- 创建一个Maven项目，在pom.xml文件中添加以下jar包
``` xml
    <!-- Java Gearman Service -->
    <dependency>
      <groupId>org.gearman.jgs</groupId>
      <artifactId>java-gearman-service</artifactId>
      <version>0.7.0-SNAPSHOT</version>
      <type>jar</type>
    </dependency>
```

该jar包包含Client和Worker的客户端API。

- Worker：EchoWorker

```java
package org.gearman.examples.echo;

import org.gearman.Gearman;
import org.gearman.GearmanFunction;
import org.gearman.GearmanFunctionCallback;
import org.gearman.GearmanServer;
import org.gearman.GearmanWorker;

/**
 * The echo worker polls jobs from a job server and execute the echo function.
 *
 * The echo worker illustrates how to setup a basic worker
 */
public class EchoWorker implements GearmanFunction {

    /** The echo function name */
    public static final String ECHO_FUNCTION_NAME = "echo";

    /** The host address of the job server */
    public static final String ECHO_HOST = "127.0.0.1";

    /** The port number the job server is listening on */
    public static final int ECHO_PORT = 4730;

    public static void main(String... args) {

        /*
         * Create a Gearman instance
         */
        Gearman gearman = Gearman.createGearman();

        /*
         * Create the job server object. This call creates an object represents
         * a remote job server.
         *
         * Parameter 1: the host address of the job server.
         * Parameter 2: the port number the job server is listening on.
         *
         * A job server receives jobs from clients and distributes them to
         * registered workers.
         */
        GearmanServer server = gearman.createGearmanServer(
                EchoWorker.ECHO_HOST, EchoWorker.ECHO_PORT);

        /*
         * Create a gearman worker. The worker poll jobs from the server and
         * executes the corresponding GearmanFunction
         */
        GearmanWorker worker = gearman.createGearmanWorker();

        /*
         *  Tell the worker how to perform the echo function
         */
        worker.addFunction(EchoWorker.ECHO_FUNCTION_NAME, new EchoWorker());

        /*
         *  Tell the worker that it may communicate with the this job server
         */
        worker.addServer(server);
    }

    @Override
    public byte[] work(String function, byte[] data,
                       GearmanFunctionCallback callback) throws Exception {

        /*
         * The work method performs the gearman function. In this case, the echo
         * function simply returns the data it received
         */
        return data;
    }

}
```

GearmanWorker的`addFunction()`方法告诉Server FUNCTION_NAME和对应的Worker。从Client传入的数据会以字节流的形式在Worker的`work()`方法中得到，并且可以在该方法中进行处理。

- Client
1. EchoClient：同步的Client

```java
package org.gearman.examples.echo;

import org.gearman.Gearman;
import org.gearman.GearmanClient;
import org.gearman.GearmanJobEvent;
import org.gearman.GearmanJobReturn;
import org.gearman.GearmanServer;

/**
 * The echo client submits an "echo" job to a job server and prints the final
 * result. It's the "Hello World" of the java-geraman-service
 *
 * The echo example illustrates how send a single job and get the result
 */
public class EchoClient {

    public static void main(String... args) throws InterruptedException {

        /*
         * Create a Gearman instance
         */
        Gearman gearman = Gearman.createGearman();

        /*
         * Create a new gearman client.
         *
         * The client is used to submit requests the job server.
         */
        GearmanClient client = gearman.createGearmanClient();

        /*
         * Create the job server object. This call creates an object represents
         * a remote job server.
         *
         * Parameter 1: the host address of the job server.
         * Parameter 2: the port number the job server is listening on.
         *
         * A job server receives jobs from clients and distributes them to
         * registered workers.
         */
        GearmanServer server = gearman.createGearmanServer(
                EchoWorker.ECHO_HOST, EchoWorker.ECHO_PORT);

        /*
         * Tell the client that it may connect to this server when submitting
         * jobs.
         */
        client.addServer(server);

        /*
         * Submit a job to a job server.
         *
         * Parameter 1: the gearman function name
         * Parameter 2: the data passed to the server and worker
         *
         * The GearmanJobReturn is used to poll the job's result
         */
        GearmanJobReturn jobReturn = client.submitJob(
                EchoWorker.ECHO_FUNCTION_NAME, ("Hello World").getBytes());

        /*
         * Iterate through the job events until we hit the end-of-file
         */
        while (!jobReturn.isEOF()) {

            // Poll the next job event (blocking operation)
            GearmanJobEvent event = jobReturn.poll();

            switch (event.getEventType()) {

                // success
                case GEARMAN_JOB_SUCCESS: // Job completed successfully
                    // print the result
                    System.out.println(new String(event.getData()));
                    break;

                // failure
                case GEARMAN_SUBMIT_FAIL: // The job submit operation failed
                case GEARMAN_JOB_FAIL: // The job's execution failed
                    System.err.println(event.getEventType() + ": "
                            + new String(event.getData()));
                default:
            }

        }

        /*
         * Close the gearman service after it's no longer needed. (closes all
         * sub-services, such as the client)
         *
         * It's suggested that you reuse Gearman and GearmanClient instances
         * rather recreating and closing new ones between submissions
         */
        gearman.shutdown();
    }
}
```

同步的Client通过GearmanClient的`submitJob(String functionName, byte[] data);`方法提交任务，并且可以得到Worker在`work()`方法中经过处理返回的数据。如果通过将任务提交后台执行的方式，只需采用`submitBackgroundJob(String functionName, byte[] data);`方法，优点是Client的执行效率会快很多，缺点是不能接收返回值。

2. EchoClientAsynchronous：异步的Client

```java
package org.gearman.examples.echo;

import org.gearman.Gearman;
import org.gearman.GearmanClient;
import org.gearman.GearmanJobEvent;
import org.gearman.GearmanJobEventCallback;
import org.gearman.GearmanJoin;
import org.gearman.GearmanServer;

/**
 * The echo client submits an "echo" job to a job server and prints the final
 * result. It's the "Hello World" of the java-geraman-service
 *
 * The echo example illustrates how send a single job and get the result
 */
public class EchoClientAsynchronous implements GearmanJobEventCallback<String> {

    public static void main(String... args) throws InterruptedException {

        /*
         * Create a Gearman instance
         */
        final Gearman gearman = Gearman.createGearman();

        /*
         * Create a new gearman client.
         *
         * The client is used to submit requests the job server.
         */
        GearmanClient client = gearman.createGearmanClient();

        /*
         * Create the job server object. This call creates an object representing
         * a remote job server.
         *
         * Parameter 1: the host address of the job server.
         * Parameter 2: the port number the job server is listening on.
         *
         * A job server receives jobs from clients and distributes them to
         * registered workers.
         */
        GearmanServer server = gearman.createGearmanServer(
                EchoWorker.ECHO_HOST, EchoWorker.ECHO_PORT);

        /*
         * Tell the client that it may connect to this server when submitting
         * jobs.
         */
        client.addServer(server);

        /*
         * Submit a job to a job server. This submit method uses an
         * asynchronous callback object to process the job's result
         *
         * Parameter 1: the gearman function name
         * Parameter 2: the data passed to the server and worker
         * Parameter 3: an attachment returned through the callback
         * Parameter 4: the callback used to process the job events
         *
         * The GearmanJoin object is used to block the current thread
         * until the end-of-file has been reached.
         */
        GearmanJoin<String> join = client.submitJob(
                EchoWorker.ECHO_FUNCTION_NAME, ("Hello World").getBytes(),
                EchoWorker.ECHO_FUNCTION_NAME, new EchoClientAsynchronous());

        /*
         * Block the current thread until all events have been processed.
         */
        join.join();

        /*
         * After the job has been completely processed. We close the service
         *
         * It's suggested that you reuse Gearman and GearmanClient instances
         * rather recreating and closing new ones between submissions
         */
        gearman.shutdown();
    }

    @Override
    public void onEvent(String attachment, GearmanJobEvent event) {

        /*
         * This method is called by the client when an event is received
         */
        switch (event.getEventType()) {
            case GEARMAN_JOB_SUCCESS: // Job completed successfully
                System.out.println(new String(event.getData()));
                break;
            case GEARMAN_SUBMIT_FAIL: // The job submit operation failed
            case GEARMAN_JOB_FAIL: // The job's execution failed
                System.err.println(event.getEventType() + ": "
                        + new String(event.getData()));
            default:
        }

    }
}
```

异步Client需实现GearmanJobEventCallback<T>，`onEvent()`监听返回事件，并接收Worker经过处理的数据。

- 运行

1. Worker Start...

执行EchoWorker的main方法，log如下

```
[gearman-1] INFO gearman - [192.168.56.102:4730] : Connected
[gearman-1] INFO gearman - [192.168.56.102:4730] : OUT : CAN_DO
[gearman-4] INFO gearman - [192.168.56.102:4730] : OUT : GRAB_JOB
[gearman-4] INFO gearman - [192.168.56.102:4730] : IN : NO_JOB
[gearman-4] INFO gearman - [192.168.56.102:4730] : OUT : PRE_SLEEP
```

Worker在等待任务提交状态。

2. Client Start...

执行EchoClient的main方法，log如下
```
[gearman-1] INFO gearman - [192.168.56.102:4730] : Connected
[gearman-1] INFO gearman - [192.168.56.102:4730] : OUT : SUBMIT_JOB
[gearman-1] INFO gearman - [192.168.56.102:4730] : IN : JOB_CREATED
[gearman-1] INFO gearman - [192.168.56.102:4730] : IN : WORK_COMPLETE
[main] INFO gearman - [192.168.56.102:4730] : Disconnected
Hello World
```

此时Client已经接收Worker直接返回的`Hello World`。再回头看看Worker的log，如下

```
[gearman-3] INFO gearman - [192.168.56.102:4730] : Connected
[gearman-3] INFO gearman - [192.168.56.102:4730] : OUT : CAN_DO
[gearman-4] INFO gearman - [192.168.56.102:4730] : OUT : GRAB_JOB
[gearman-5] INFO gearman - [192.168.56.102:4730] : IN : NO_JOB
[gearman-5] INFO gearman - [192.168.56.102:4730] : OUT : PRE_SLEEP
[gearman-4] INFO gearman - [192.168.56.102:4730] : IN : NOOP
[gearman-5] INFO gearman - [192.168.56.102:4730] : OUT : GRAB_JOB
[gearman-4] INFO gearman - [192.168.56.102:4730] : IN : JOB_ASSIGN
[gearman-3] INFO gearman - [192.168.56.102:4730] : OUT : WORK_COMPLETE
[gearman-4] INFO gearman - [192.168.56.102:4730] : OUT : GRAB_JOB
[gearman-4] INFO gearman - [192.168.56.102:4730] : IN : NO_JOB
[gearman-4] INFO gearman - [192.168.56.102:4730] : OUT : PRE_SLEEP
```

Worker执行完任务后，继续等待接收新的任务。

