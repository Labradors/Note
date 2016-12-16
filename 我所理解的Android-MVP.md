---
title: 我所理解的Android MVP
date: 2016-08-30 19:51:11
tags: Android架构
categories: Android
---

&nbsp;&nbsp;&nbsp;&nbsp; 刚写完一个有关一个Android MVP模式的Demo，Demo地址在[AndroidDesignPattern](https://github.com/BosCattle/AndroidDesignPattern),记录一下MVP的使用方式，个人觉得MVC还是很好理解的。这篇文章主要记录了MVP是什么？作用是什么？MVP相对MVC的差别。最重要的是讲MVP的使用方式。<!--more-->

## MVP与MVC

- MVC

 *M代表Model层，V代表视图层，C是控制器层，具体解释可以查看[维基百科](https://zh.wikipedia.org/wiki/MVC)，在没有任何架构的情况下，我们的Activity作为控制器层与视图层的结合，页面逻辑简单还好，如果页面逻辑复杂，在几千行代码中找一个bug，或者版本升级，那滋味，你想想就爽啊！*
 ![MVC](http://7xk0q3.com1.z0.glb.clouddn.com/QQ20160831-0@2x.png)

- MVP

 *MVP的出现解决了Activity或者Fragment下，控制器层与视图层之间的问题，MVP用Presenter层作为逻辑层，将逻辑层于视图层分离，通过接口的方式，让View层只做UI方面的工作，而Presenter则对数据进行处理，将View层与Model层隔离。如图：*
 ![MVP](http://7xk0q3.com1.z0.glb.clouddn.com/QQ20160831-1@2x.png)

 ## MVP实现方式

 ### 定义与UI相关的接口
 &nbsp;&nbsp;&nbsp;&nbsp; 比如设置某个TextView的值，加载图片等等。代码如下:
 ```Java
 interface View {

     void setAvatar(String url);

     void setUserName(String name);

     void error();
   }
 ```

 ### 定义与数据相关的接口
 &nbsp;&nbsp;&nbsp;&nbsp; 在通过网络，本地获取数据，获取数据后需要做的工作，以接口方式定义。如下:

 ```Java
 public interface Presenter {

   void loadGithubData(String userName);

   interface DataStatus{

     void error(Throwable varThrowable);

     void success(Users varUsers);
   }
 }
 ```

 ### 实现Presenter
&nbsp;&nbsp;&nbsp;&nbsp; 实现Presenter接口，完善方法，并在方法中通过View更新数据

```Java
public class LoadGithubPresenter
    implements LoadGithubContract.Presenter, LoadGithubContract.Presenter.DataStatus {

  public static final String TAG = LoadGithubPresenter.class.getSimpleName();
  private LoadGithubContract.View mView;

  public LoadGithubPresenter(LoadGithubContract.View varView) {
    mView = varView;
    Log.d(TAG, "LoadGithubPresenter: 初始化Presenter");
  }

  @Override public void loadGithubData(String userName) {
    Subscription s = GitHubApi.getContributors(userName)
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(this::success, this::error);
  }

  @Override public void error( Throwable var) {
    Log.d(TAG, "error: "+var.getMessage());
    mView.error();
  }

  @Override public void success(Users varUsers) {
    mView.setUserName(varUsers.login);
    mView.setAvatar(varUsers.avatar_url);
  }
}

```

 ### 在Activity中使用
 &nbsp;&nbsp;&nbsp;&nbsp;在Activity中实现View,并且初始化，调用数据加载函数。ok，等着见证最后的结果:

 ```Java

 public class MvpActivity extends AppCompatActivity implements LoadGithubContract.View {

   public static final String TAG = MvpActivity.class.getSimpleName();

   private SimpleDraweeView mSimpleDraweeView;
   private TextView mTextView;
   private LoadGithubContract.Presenter mPresenter;

   @Override protected void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     setContentView(R.layout.activity_mvp);
     mSimpleDraweeView = (SimpleDraweeView) findViewById(R.id.my_image_view);
     mTextView = (TextView) findViewById(R.id.my_name_view);
     mPresenter = new LoadGithubPresenter(MvpActivity.this);
   }

   @Override protected void onResume() {
     super.onResume();
     mPresenter.loadGithubData("jiangTaoQuite");
   }

   @Override public void setAvatar(String url) {
     mSimpleDraweeView.setImageURI(url);
   }

   @Override public void setUserName(String name) {
     mTextView.setText(name);
   }

   @Override public void error() {
     Toast.makeText(this,"获取数据错误",Toast.LENGTH_LONG).show();
   }
 }
 ```
- 欢迎指正以及提出建议
