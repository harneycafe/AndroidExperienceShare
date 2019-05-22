## Binder简单梳理
Binder被设计主要是为了解决跨进程通信的问题，关于Binder的文章有很多，深入原理的话大部分都能从Java层讲到C++层，但是我看一遍忘一遍，并且这些原理在App开发中也用不到，所以我感觉，只要熟记它的架构模型，在必要的时候再去看细节，对它有足够整体的认识就好。
### Binder架构模型
Binder属于C-S架构，它有几个重要的角色，分别为Client、Server、ServiceManager。Binder里面的Client和Server是相对的，谁发消息谁就是Client，谁接收消息谁就是Server(不太严谨)。<br/>

每个Server都需要注册在ServiceManager中才能在IPC过程中被访问到，下面贴出一张图：
![image](http://note.youdao.com/yws/res/165/WEBRESOURCE7c3a4ce28db1f3886506cb82fd1e188e)

### Binder通信过程
#### Binder通信需要满足几步：
* Server需要注册在ServiceManager容器中
* Client若要调用Server的方法，需要先获取Server对象，但是ServiceManager不会把真正的Server返回给Client，而是把Server的一个代理对象Proxy返回给Client
* Client调用Proxy的方法，ServerManager会帮他调用Server的方法，并把结果返回给Client。
### AIDL
了解了这些Binder知识，我们再去看AIDL。AIDL是Binder的延伸，Android系统中很多服务都是基于AIDL，如剪切板等。对App开发人员来说，AIDL应该算是需要掌握的一个技术点吧！AIDL中有几个非常重要的类，如下：
* IBinder
* IInterface
* Binder
* Proxy
* Stub

创建AIDL文件的方法我就不说了。[这里附上AIDL使用地址](https://developer.android.com/guide/components/aidl?hl=zh-cn)

这里我们定义一个AIDL文件如下，里面只有一个add()方法：
```
// ITestAidlInterface.aidl
package com.example.xiashengming.plugintest;

// Declare any non-default types here with import statements

interface ITestAidlInterface {

    void add(int a, int b);

}
```
AndroidStudio通过AIDL文件会给我们生成继承自IInterface的接口，如下：
```
public interface ITestAidlInterface extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.example.xiashengming.plugintest.ITestAidlInterface {
        private static final java.lang.String DESCRIPTOR = "com.example.xiashengming.plugintest.ITestAidlInterface";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.example.xiashengming.plugintest.ITestAidlInterface interface,
         * generating a proxy if needed.
         */
        public static com.example.xiashengming.plugintest.ITestAidlInterface asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.example.xiashengming.plugintest.ITestAidlInterface))) {
                return ((com.example.xiashengming.plugintest.ITestAidlInterface) iin);
            }
            return new com.example.xiashengming.plugintest.ITestAidlInterface.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            java.lang.String descriptor = DESCRIPTOR;
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(descriptor);
                    return true;
                }
                case TRANSACTION_add: {
                    data.enforceInterface(descriptor);
                    int _arg0;
                    _arg0 = data.readInt();
                    int _arg1;
                    _arg1 = data.readInt();
                    this.add(_arg0, _arg1);
                    reply.writeNoException();
                    return true;
                }
                default: {
                    return super.onTransact(code, data, reply, flags);
                }
            }
        }

        private static class Proxy implements com.example.xiashengming.plugintest.ITestAidlInterface {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            @Override
            public void add(int a, int b) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeInt(a);
                    _data.writeInt(b);
                    mRemote.transact(Stub.TRANSACTION_add, _data, _reply, 0);
                    _reply.readException();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }
        }

        static final int TRANSACTION_add = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
    }

    public void add(int a, int b) throws android.os.RemoteException;
}
```
ITestAidlInterface是系统为我们生成的接口，其内部有一个抽象类Stub，Stub内部有内部类Proxy<br/>
（1）从Client进程来看，asInterface(IBinder obj)方法的作用是判断传过来的IBinder对象是否是同一个进程，如果是则直接转换使用，如果不是则把IBinder包装成一个Proxy对象，间接使用。<br/>
（2）Proxy在自己的add(int a,int b)方法中使用Parcelable来准备数据，把函数名称、参数写入 _data 中，让 _reply 接受函数返回值，最后通过IBinder的transact()方法，就可以把数据传给Binder的Server端了。<br/>
（3）Server端通过Stub的onTrancest()方法接收Client进程传过来的方法名称、参数等，然后调用真正的add()方法把结果返回。
