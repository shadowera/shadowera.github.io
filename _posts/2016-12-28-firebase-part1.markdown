---
layout:     post
title:      "Firebase-part1-Authentication"
subtitle:   " A new way to build apps"
date:       2016-12-28 12:00:00
author:     "shadowera"
header-img: "img/about-bg.jpg"
catalog: true
tags:
    - android
    - Firebase
---
##  首先
当app发展到本地以外，就不得不牵扯到服务器了，当然这对于大公司来说并不是什么大问题，服务器有后台人员维护，网站有前端
人员负责，移动端也有对应人员搞定。但是服务器就一个（或一组，一间etc），其中的数据要联动到web，Android，iOS等不同平台上，
可能就会有不同的差异，开发工具包自然是一点，从服务器拉取的方式和传送数据的格式可能也会有不同。 
对于小型开发团队或者独立开发者来说就更痛苦了，在开发应用的同时还需要抽身去创建和维护服务器，即使现在已经有很多的云服务供
应商，但是这点还是避免不了。 

Firebase作为提供实时后端数据库的公司，它可以帮助快速地开发网页端和移动端的应用，自从Google收购了之后，在今年的IO大会上
大放异彩，它在使原来的服务更加方便之外，还加入了Google的各种新元素，包括远程配置，云上调试，崩溃报告，动态链接等等，基本上所有的
功能都只要几行代码就能完成，而惊喜的是，这些工具都是跨平台的，这意味着代码的移植变得很简单。
 
另外Firebase Cloud Messaging也是Google Cloud Messaging的扩展版，随着Google对中国的兴趣越来越大，将来说不定还
是会回归的，再加上Android N中加强版的Doze mode强调了GCM的重要性，因此Firebase在未来Android应用的作用将不可小觑。

下面几篇博客分别来介绍firebase的功能及使用。

## 身份认证 Authentication

> 用户使用app的第一步，就是身份认证，包括注册，登陆还有登出功能。Firebase的Auth模块就是对应身份认证这一点的。Firebase的登陆方式除了基本的邮箱+密码方式，还有第三方账号的登陆方式，包括Google，Facebook，twitter和Gihub，除此之外，Firebase也允许开发者自定义登陆方式 在启用某种登录方式时，需要在Firebase控制台的Auth模块中，在”登录方法“标签页下进行对应的启用

firebase目前支持的登录方式有：

* 邮箱密码
* google
* github
* facebook
* 自定义登录

### 邮箱+密码
首先先来看最基本的邮件+密码方式。 
为了使用身份认证功能，需要为项目添加对firebase-auth库的依赖：
```
dependencies {
    // ...
    compile 'com.google.firebase:firebase-auth:9.0.0'
}
```
FirebaseAuth是Firebase身份认证的入口，就是由它来完成身份认证的各种功能 
FirebaseAuth使用了单例模式，应用的FirebaseAuth对象通过FirebaseAuth.getInstance()获得（说是单例但其实不是很准确，因为getInstance()方法的说明是”返回默认的FirebaseApp实例所对应的FirebaseAuth实例“，该方法也有一个接收FirebaseApp参数的重载版本，但是应用通常不需要和FirebaseApp交互，所使用的FirebaseApp实例就是默认的版本，所以狭义地来说是”单例的“）
#### 注册用户
FirebaseAuth的注册方法是：

```java
mAthu.createUserWithEmailAndPassword(
             email.getText().toString(),
             password.getText().toString()
         )
         .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
             @Override
             public void onComplete(@NonNull Task<AuthResult> task) {
                 if (!task.isSuccessful()) {
                     // ...
                 } else {
                     // ...
                 }
             }
         });
```
其中这两个参数分别是邮件和密码。方法将会立即返回一个Task对象，Task类是用于处理Firebase异步操作的，必须使用一个TResult作为类型参数，表示此次操作的结果。
对于身份，认证这里最终会返回一个AuthResult对象，表示此次认证操作的结果 
为了监听认证操作的结果，需要为这个Task对象添加一个OnCompleteListener .

当注册成功后，用户会自动登录，即FirebaseAuth会相应地发生改变，为了监听FirebaseAuth状态的变化，可以给其添加一个AuthStateListener。需要注意的是，因为FirebaseAuth是单例的，而且可能会在多个多个活动中使用到FirebasAuth（比如登入，登出，注册等等），所以需要在活动start和stop的时候分别进行add和remove监听器，以防发生冲突：

```java
private FirebaseAuth.AuthStateListener mAuthListener;

@Override
protected void onCreate(Bundle savedInstanceState) {

    mAuthListener = new FirebaseAuth.AuthStateListener() {
        @Override
        public void onAuthStateChanged(@NonNull FirebaseAuth firebaseAuth) {
            FirebaseUser user = firebaseAuth.getCurrentUser();
            if (user != null) {
                // 用户已登入或发生改变
            } else {
                // 用户已注销
            }
        }
    };
}

@Override
public void onStart() {
    super.onStart();
    mAuth.addAuthStateListener(mAuthListener);
}

@Override
public void onStop() {
    super.onStop();
    if (mAuthListener != null) {
        mAuth.removeAuthStateListener(mAuthListener);
    }
}
```
通过getCurrentUser()得到对应的FirebaseUser对象，它提供了当前用户的各种用户信息，例如：

方法	|作用
------|------
String getUid()|	返回用户的uid
String getDisplayName()	|返回用户名
String getEmail()	|返回用户email
String getPhotoUrl()|	返回用户头像
boolean isAnonymous()|	是否匿名
Task updatePassword(String)|	更新密码
Task updateEmail(String)|	更新email

#### 登录

```java
mAuth.signInWithEmailAndPassword(email, password)
                .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        Log.d(TAG, "signIn:onComplete:" + task.isSuccessful());
                        hideProgressDialog();

                        if (task.isSuccessful()) {
                            onAuthSuccess(task.getResult().getUser());
                        } else {
                            Toast.makeText(SignInActivity.this, "Sign In Failed",
                                    Toast.LENGTH_SHORT).show();
                        }
                    }
                });
```

####  注销

```java
FirebaseAuth.getInstance().signOut();
```

### Google账号登录
Google Play Services SDK中也提供了使用Google账号的验证方式 
为了使用这个功能，除了必须的firebase-auth库，还需要为项目添加依赖：

```java
dependencies {
    compile 'com.google.android.gms:play-services-auth:9.0.0'
}
```

#### 配置Google Sign In方案
GoogleSignInOptions

GoogleSignInOptions将为后面的登录设定一些选项，它使用建造者模式来产生一个对象。通常会使用默认的DEFAULT_SIGN_IN，然后再为其添加一些额外选项。注意，如果要使用后端服务器或者Google Sign-In进行认证，必须为其提供服务器的OAuth2.0客户端ID，对于Google Sign-In，在将google-services.json添加到项目中并sync完成后，XML中将会自动地加入一系列地配置信息，其中就包括默认的服务器客户端ID：

```java
GoogleSignInOptions gso = new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
                .requestIdToken(getString(R.string.default_web_client_id))
                .requestEmail()
                .build();
```

GoogleApiClient

GoogleSignInOptions是用来提供登录选项的，那谁来负责登录呢？就是GoogelApiClient了，关于GoogleApiClient的生成比较复杂，根据官方的建议，一般的写法是：

```java
mGoogleApiClient = new GoogleApiClient.Builder(this)
        .enableAutoManage(this /* FragmentActivity */, this /* OnConnectionFailedListener */)
        .addApi(Auth.GOOGLE_SIGN_IN_API, gso)
        .build();
```

需要注意的是，enableAutoManage方法是用一个FragmentActivity来自动管理GoogleApiClient，一般是当前所在的活动，该活动必须继承于FragmentActivity或AppCompatActivity，否则就必须手动管理GoogleApiClient的连接周期。另一个参数则是用来监听连接失败时间的监听器 
在addApi方法中，传入之前创建的GoogleSignInOptions对象

Google推荐使用专门的Button来作登录按钮： 

![google button](https://developers.google.com/identity/images/btn_google_signin_light_normal_web.png)
这个控件就是SignInButton。它提供了setSize()，setColorScheme()和setScopes()来为外观进行定制

需要使用到的三个部件介绍完了，接下来就是Google Sign-In的登录流程：
1. SignInButton点击事件

```java
Intent signInIntent = Auth.GoogleSignInApi.getSignInIntent(mGoogleApiClient);
startActivityForResult(signInIntent, RC_SIGN_IN);
```

2. 设置回调逻辑

```java
@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    // Result returned from launching the Intent from GoogleSignInApi.getSignInIntent(...);
    if (requestCode == RC_SIGN_IN) {
        GoogleSignInResult result = Auth.GoogleSignInApi.getSignInResultFromIntent(data);
        if (result.isSuccess()) {
            // ...
        } else {
            // ...
        }
    }
}
```

Google Sign-In的介绍就大概是这样了 
至于第三方账号的支持，如Facebook，Twitter和Github，流程基本都是根据对应SDK获得对应的令牌对象，然后根据令牌创建AuthCredential，最后FirebaseAuth调用signInWithCredential()方法进行登录即可

至于服务器自定义认证的方式涉及到服务器的配置，所以介绍的篇幅会比较长，这里就不作介绍，等到以后有机会再写吧 
Firebase的身份认证模块就介绍到这了，后面将会持续对其他模块进行学习




