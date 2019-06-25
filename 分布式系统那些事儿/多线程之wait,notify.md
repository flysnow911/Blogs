wait要在同步块中调用，因为wait逻辑是释放wait方法调用者的monitor锁，既然释放，那必须得到。要保证得到，就要同步。
所以必须在sycronize中执行。
notify唤醒那些在等待该对象monitor锁的线程。多个线程在等待的话，那么就唤醒队列中的第一个。
notifyAll唤醒那些在等待该对象monitor锁的所有线程。

在下面例子中，对象锁是obj, 消费线程先执行，获得obj锁，然后发现需要的商品没生产出来，就等待，调用obj.wait()表示释放obj的monitor锁，进入obj的monitor锁获取等待队列。
生产都生产结束后，需要唤醒消费者，就调用obj的notify,表示唤醒等待obj对象monitor锁的线程。


public class ThreadTest {

    static final Object obj = new Object();  //对象锁

    private static boolean flag = false;

    public static void main(String[] args) throws Exception {

        Thread consume = new Thread(new Consume(), "Consume");
        Thread produce = new Thread(new Produce(), "Produce");
        consume.start();
        Thread.sleep(1000);
        produce.start();

        try {
            produce.join();
            consume.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    // 生产者线程
    static class Produce implements Runnable {

        @Override
        public void run() {

            synchronized (obj) {
                System.out.println("进入生产者线程");
                System.out.println("生产");
                try {
                    TimeUnit.MILLISECONDS.sleep(2000);  //模拟生产过程
                    flag = true;
                    obj.notify();  //通知消费者
                    TimeUnit.MILLISECONDS.sleep(1000);  //模拟其他耗时操作
                    System.out.println("退出生产者线程");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    //消费者线程
    static class Consume implements Runnable {

        @Override
        public void run() {
            synchronized (obj) {
                System.out.println("进入消费者线程");
                System.out.println("wait flag 1:" + flag);
                while (!flag) {  //判断条件是否满足，若不满足则等待
                    try {
                        System.out.println("还没生产，进入等待");
                        obj.wait();
                        System.out.println("结束等待");
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("wait flag 2:" + flag);
                System.out.println("消费");
                System.out.println("退出消费者线程");
            }

        }
    }
}


·