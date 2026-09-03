<img width="1222" height="706" alt="image" src="https://github.com/user-attachments/assets/fcc1bc8c-5602-4116-a117-8f0242aa9c1a" />
## Dead lock
<img width="1123" height="773" alt="image" src="https://github.com/user-attachments/assets/e1dddc86-f35c-45fb-84b1-03ee10497cab" />
<img width="1184" height="848" alt="image" src="https://github.com/user-attachments/assets/f9254fcf-448a-461f-8f47-202304e63067" />

## Dead lock occure and my codeis 
'''
import jdk.jfr.StackTrace;

import javax.swing.*;
import java.math.BigInteger;
import java.util.LinkedList;
import java.util.Map;
import java.util.Objects;
import java.util.Queue;
import java.util.concurrent.Callable;

class ServiceA{
    private int resoureA =100;
    public synchronized void proccess(ServiceB serviceB)
    {
        System.out.println(" process START by  "+Thread.currentThread().getName()+"inside serviceA");
        System.out.println("resurecr b need insedes A "+serviceB.getResoureB());
        System.out.println(" process end by  "+Thread.currentThread().getName()+"inside serviceA");
    }


   public synchronized int getResoureA()
    {
        return resoureA;
    }


}
class ServiceB{
    private int resoureB =200;
    public synchronized void proccess(ServiceA serviceA)
    {
        System.out.println(" process START by  "+Thread.currentThread().getName()+"inside serviceB");
        System.out.println("resurecr A need insedes B "+serviceA.getResoureA());
        System.out.println(" process end by  "+Thread.currentThread().getName()+"inside serviceB");
    }


    public synchronized int getResoureB()
    {
        return resoureB;
    }
}

public class Main {

    public  static  void main(String[] args) throws InterruptedException {
    ServiceA serviceA = new ServiceA();
    ServiceB serviceB  = new ServiceB();
        Thread T1 = new Thread(()->{
        serviceA.proccess(serviceB);


        });
        Thread T2 = new Thread(()->{
           serviceB.proccess(serviceA);

        });
    T1.start();
    T2.start();


        System.out.println("main task end.....");

}

}
```



