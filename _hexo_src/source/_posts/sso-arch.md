---
title: 单点登录解决方案
date: 2017-12-05 10:27:26
tags:
categories: 系统架构
---

## 什么是SSO
Web应用系统的演化总是从简单到复杂，从单功能到多功能模块再到多子系统方向发展。当前的大中型Web互联网应用基本都是多系统组成的应用群，由多个web系统协同为用户提供服务。
多系统应用群，必然意味着各系统间既相对独立，又保持着某种联系。
独立，意味着给用户提供一个相对完整的功能服务，比如C2C商城，比如B2C商城。联系，意味着从用户角度看，不管企业提供的服务如何多样化、系列化，在用户看来，仍旧是一个整体，用户体验不能受到影响。
譬如用户的账号管理，用户应该有一个统一账号，不应该让用户在每个子系统分别注册、分别登录、再分别登出。系统的复杂性不应该让用户承担。登录用户使用系统服务，可以看做是一次用户会话过程。在单Web应用中，用户登录、登录状态判断、用户登出等操作，已有很常规的解决方案实现。
在多系统应用群中，这个问题就变得有些复杂，以前本不是问题的问题，现在可能就变成了一个重大技术问题。我们要用技术手段，屏蔽系统底层本身的技术复杂性，给用户提供自然超爽的用户体验。
这就是的单点登录问题，即SSO(Single Sign On)。我们这里主要讨论Web系统，应该叫Web SSO。

## 实际案例
例如阿里系统种类繁多，典型系统应用www.taobao.com 淘宝应用、www.tmall.com 天猫应用、www.alitrip.com 阿里旅游。这些应用，当用户访问时，都需要登录。当我在一个应用如淘宝上登录后，再访问阿里旅游、天猫等其它系统，我们发现，系统都显示已登录状态。当在任意一系统退出登录后，再刷新访问其它系统，均已显示登出状态。可以看出，阿里实现了SSO。实际上，几乎所有提供复杂服务的互联网公司，都实现了SSO，如阿里、百度、新浪、网易、腾讯、58...SSO问题，是大中型Web应用经常碰到的问题，是Java架构师需要掌握的必备技能之一，中高级以上Web工程师都应对它有个了解。

## SSO技术难点

SSO有啥技术难点？为什么我们不能像解决单Web应用系统登录那样自然解决？为说清楚这一问题，我们得先了解下单应用系统下，用户登录的解决方案。

### 单应用系统登录
我们讨论的应用是Web应用，大家知道，对于Web应用，系统是Browser/Server架构，Browser和Server之间的通信协议是HTTP协议。HTTP是一个无状态协议。即对服务器来说，每次收到的浏览器HTTP请求都是单一独立的，服务器并不考虑两次HTTP请求是否来自同一会话，即HTTP协议是非连接会话状态协议。对于Web应用登录，意味着登录成功后的后续访问，可以看做是登录用户和服务端的一次会话交互过程，直到用户登出结束会话。如何在非连接会话协议之上，实现这种会话的管理？ 我们需要额外的手段。通常有两种做法，一种是通过使用HTTP请求参数传递，这种方式对应用侵入性较大，一般不使用。另一种方式就是通过cookie。cookie是HTTP提供的一种机制，cookie代表一小撮数据。服务端通过HTTP响应创建好cookie后，浏览器会接收下来，下次请求会自动携带上返回给服务端。利用这个机制，我们可以实现应用层的登录会话状态管理。例如我们可以把登录状态信息保存在cookie中，这是客户端保存方式。由于会话信息在客户端，需要维护其安全性、需要加密保存、携带量会变大，这样会影响http的处理效率，同时cookie的数据携带量也有一定的限制。比较好的方式是服务端保存，cookie只保存会话信息的句柄。即在登录成功后，服务端可以创建一个唯一登录会话，并把会话标识ID通过cookie返回给浏览器，浏览器下次访问时会自动带上这个ID，服务端根据ID即可判断是此会话中的请求，从而判断出是该用户，这种操作直到登出销毁会话为止。令人高兴的是，我们使用的Web应用服务器一般都会提供这种会话基础服务，如Tomcat的Session机制。也就是说，应用开发人员不必利用Cookie亲自代码实现会话的创建、维护和销毁等整个生命周期管理，这些内容服务器Session已经提供好了，我们只需正确使用即可。当然，为了灵活性和效率，开发人员也可直接使用cookie实现自己的这种会话管理。对于Cookie，处于安全性考虑，它有一个`作用域问题`，这个作用域由属性`Domain`和`Path`共同决定的。也就是说，如果浏览器发送的请求不在此Cookie的作用域范围内，请求是不会带上此Cookie的。Path是访问路径，我们可以定义/根路径让其作用所有路径，Domain就不一样了。我们不能定义顶级域名如`.com`，让此Cookie对于所有的com网站都起作用，最大范围我们只能定义到二级域名如`.taobao.com`，而通常，企业的应用群可能包含有多个二级域名，如taobao.com、tmail.com、alitrip.com等等。这时，解决单系统会话问题的Cookie机制不起作用了，多系统不能共享同一会话，这就是问题的所在！

当然，有的同学会说：我把所有的应用统一用三级域名来表示，如a.taobao.com、b.taobao.com、c.taobao.com或干脆用路径来区分不同的应用如www.taobao.com\a、www.taobao.com\b、www.taobao.com\c，这样cookie不就可以共享了么？事实是成立的，但现实应用中，多域名策略是普遍存在的，也有商业角度的考虑，这些我们必须要面对。退一步讲，即使cookie可以共享了，服务端如何识别处理这个会话？这时，我们是不能直接使用服务器所提供的Session机制的，Session是在单一应用范围内，共享Session需要特殊处理。更复杂的情况是，通常这些子系统可能是异构的，session实现机制并不相同，如有的是Java系统，有的是PHP系统。共享Session对原系统入侵性很大。至此，SSO技术问题这里讲清楚了。那我们有没有更好的通用解决方案？答案肯定是有的，但比较复杂，这也是我们专题讨论的理由。总体来说，我们需要一个中央认证服务器，来统一集中处理各子系统的登录请求。

### SSO实现基本思路
单Web应用登录，主要涉及到认证（用户名密码）、授权（权限定义）、会话建立（Cookie&Session）、取消会话（删除Session）等几个关键环节。推广到多系统，每个系统也会涉及到认证、授权、会话建立取消等工作。那我们能不能把每个系统的认证工作抽象出来，放到单独的服务应用中取处理，是不是就能解决单点登录问题？

思考方向是正确的，我们把这个统一处理认证服务的应用叫认证中心。当用户访问子系统需要登录时，我们把它引到认证中心，让用户到认证中心去登录认证，认证通过后返回并告知系统用户已登录。当用户再访问另一系统应用时，我们同样引导到认证中心，发现已经登录过，即返回并告知该用户已登录

### 三大关键问题

#### 登录信息传递问题
应用系统将登录请求转给认证中心，这个很好解决，我们一个HTTP重定向即可实现。现在的问题是，用户在认证中心登录后，认证中心如何将消息转回给该系统？这是在单web系统中不存在的问题。我们知道HTTP协议传递消息只能通过请求参数方式或cookie方式，cookie跨域问题不能解决，我们只能通过URL请求参数。我们可以将认证通过消息做成一个令牌(token)再利用HTTP重定向传递给应用系统。但现在的关键是：该系统如何判断这个令牌的真伪？如果判断这个令牌确实是由认证中心发出的，且是有效的？我们还需要应用系统和认证中心之间再来个直接通信，来验证这个令牌确实是认证中心发出的，且是有效的。由于应用系统和认证中心是属于服务端之间的通信，不经过用户浏览器，相对是安全的。

用户首次登录时流程如下：

1. 用户浏览器访问系统A需登录受限资源。 
2. 系统A发现该请求需要登录，将请求重定向到认证中心，进行登录。 
3. 认证中心呈现登录页面，用户登录，登录成功后，认证中心重定向请求到系统A，并附上认证通过令牌。 
4. 系统A与认证中心通信，验证令牌有效,证明用户已登录。 
5. 系统A将受限资源返给用户。 

已登录用户首次访问应用群中系统B时：

1. 浏览器访问另一应用B需登录受限资源。
2. 系统B发现该请求需要登录，将请求重定向到认证中心，进行登录。
3. 认证中心发现已经登录，即重定向请求响应到系统B，附带上认证令牌。
4. 系统B与认证中心通信，验证令牌有效,证明用户已登录。
5. 系统B将受限资源返回给客户端。

#### 登录状态判断问题
用户到认证中心登录后，用户和认证中心之间建立起了会话，我们把这个会话称为全局会话。当用户后续访问系统应用时，我们不可能每次应用请求都到认证中心去判定是否登录，这样效率非常低下，这也是单Web应用不需要考虑的。我们可以在系统应用和用户浏览器之间建立起局部会话，局部会话保持了客户端与该系统应用的登录状态，局部会话依附于全局会话存在，全局会话消失，局部会话必须消失。用户访问应用时，首先判断局部会话是否存在，如存在，即认为是登录状态，无需再到认证中心去判断。如不存在，就重定向到认证中心判断全局会话是否存在，如存在，按1提到的方式通知该应用，该应用与客户端就建立起它们之间局部会话，下次请求该应用，就不去认证中心验证了。

#### 登出问题
用户在一个系统登出了，访问其它子系统，也应该是登出状态。要想做到这一点，应用除结束本地局部会话外，还应该通知认证中心该用户登出。认证中心接到登出通知，即可结束全局会话，同时需要通知所有已建立局部会话的子系统，将它们的局部会话销毁。这样，用户访问其它应用时，都显示已登出状态。整个登出流程如下：

1. 客户端向应用A发送登出Logout请求。
2. 应用A取消本地会话，同时通知认证中心，用户已登出。
3. 应用A返回客户端登出请求。
4. 认证中心通知所有用户登录访问的应用，用户已登出。

### 实现SSO
实现分成两大部分，一个是SSO Server，代表认证中心，另一个是SSO Client，代表使用SSO系统应用的登录登出组件。

#### 登录令牌token实现
前面我们讨论了，系统把用户重定向导向认证中心并登录后，认证中心要把登录成功信息通过令牌方式告诉给应用系统。认证中心会记录下来自某个应用系统的某个用户本次通过了认证中心的认证所涉及的基本信息，并生成一个登录令牌token，认证中心需要通过URL参数的形式把token传递回应用系统，由于经过客户端浏览器，故令牌token的安全性很重要。因此令牌token的实现要满足三个条件：

首先，token具有唯一性，它代表着来自某应用系统用户的一次成功登录。我们可以利用java util包工具直接生成一个32位唯一字符串来实现。
```java
String token = UUID.randomUUID().toString();
```
同时，我们定义一个javabean， TokenInfo 来承载token所表示的具体内容，即某个应用系统来的某个用户本次通过了认证中心
```java
public class TokenInfo { 
    private int userId; //用户唯一标识ID 
    private String username; //用户登录名 
    private String ssoClient; //来自登录请求的某应用系统标识 
    private String globalId; //本次登录成功的全局会话sessionId ... 
}
```
token和tokenInfo形成了一个`<key,value>`形式的键值对，后续应用系统向认证中心验证token时还会用到。其次，token存在的有效期间不能过长，这是出于安全的角度，例如token生存最大时长为60秒。我们可以直接利用redis特性来实现这一功能。redis本质就是`<key,value>`键值对形式的内存数据库，并且这个键值对可以设置有效时长。第三，token只能使用一次，用完即作废，不能重复使用。这也是保证系统安全性。我们可以定义一个TokenUtil工具类，来实现`<token,tokenInfo>`键值对在redis中的操作，主要接口如下：
```java
public class TokenUtil { 
    ... 
    // 存储临时令牌到redis中，存活期60秒 
    public static void setToken(String tokenId, TokenInfo tokenInfo){ ... } 
    //根据token键取TokenInfo 
    public static TokenInfo getToken(String tokenId){ ... } 
    //删除某个 token键值 
    public static void delToken(String tokenId){ ... } 
}
```
#### 全局会话和本地会话的实现
用户登录成功后，在浏览器用户和认证中心之间会建立全局会话，浏览器用户与访问的应用系统之间，会建立本地局部会话。为简便可以使用web应用服务器(如tomcat)提供的session功能来直接实现。这里需要注意的是，我们需要根据会话ID(即sessionId)能访问到这个session。因为根据前面登出流程说明，认证中心的登出请求不是直接来自连接的浏览器用户，可能来自某应用系统。认证中心也会通知注册的系统应用进行登出。这些请求，都是系统之间的交互，不经过用户浏览器。系统要有根据sessionId访问session的能力。同时，在认证中心中，还需要维护全局会话ID和已登录系统本地局部会话ID的关系，以便认证中心能够通知已登录的系统进行登出处理。为了安全，目前的web应用服务器，如tomcat，是不提供根据sessionId访问session的能力的，那是容器级范围内的能力。我们需要在自己的应用中，自己维护一个sessionId和session直接的对应关系，我们把它放到一个Map中，方便需要时根据sessionId找到对应的session。同时，我们借助web容器提供的session事件监听能力，程序来维护这种对应关系。认证中心涉及到两个类，GlobalSessions和GlobalSessionListener，相关代码如下：
```java
public class GlobalSessions { 
    //存放所有全局会话 
    private static Map<String, HttpSession> sessions = 
                            new HashMap<String,HttpSession>(); 
    public static void addSession(String sessionId, HttpSession session) { 
        sessions.put(sessionId, session); 
    } 
    public static void delSession(String sessionId) { 
        sessions.remove(sessionId); 
    } 
    //根据id得到session 
    public static HttpSession getSession(String sessionId) { 
        return sessions.get(sessionId); 
    }
} 
public class GlobalSessionListener implements HttpSessionListener { 
    public void sessionCreated(HttpSessionEvent httpSessionEvent) { 
            GlobalSessions.addSession( 
                httpSessionEvent.getSession().getId(), 
                httpSessionEvent.getSession()); 
    } 
    public void sessionDestroyed(HttpSessionEvent httpSessionEvent) { 
        GlobalSessions.delSession(httpSessionEvent.getSession().getId()); 
    } 
}
```
SSO Client对应的是LocalSessions和LocalSessionListener,实现方式同上。

#### 应用系统和认证中心之间的通信
根据SSO实现流程，应用系统和认证中心之间需要直接通信。如应用系统需要向认证中心验证令牌token的真伪，应用系统通知认证中心登出，认证中心通知所有已注册应用系统登出等。这是Server之间的通信，如何实现呢？我们可以使用HTTP进行通信，返回的消息应答格式可采用JSON格式。
Java的net包，提供了http访问服务器的能力。这里，我们使用apache提供的一个更强大的开源框架，httpclient，来实现应用系统和认证中心之间的直接通信。JSON和JavaBean之间的转换，目前常用的有两个工具包，一个是json-lib，还有一个是Jackson，Jackson效率较高，依赖包少，社区活跃度大，这里我们使用Jackson这个工具包。
如应用系统向认证中心发送token验证请求的代码片段如下：

```java
//向认证中心发送验证token请求 
String verifyURL = "http://" + server 
                   + PropertiesConfigUtil.getProperty("sso.server.verify"); 
HttpClient httpClient = new DefaultHttpClient(); 
//serverName作为本应用标识 
HttpGet httpGet = new HttpGet(verifyURL + "?token=" + token 
                  + "&localId=" + request.getSession().getId()); 

try{ 
    HttpResponse httpResponse = httpClient.execute(httpGet); 
    int statusCode = httpResponse.getStatusLine().getStatusCode(); 
    if (statusCode == HttpStatus.SC_OK) { 
        String result = EntityUtils.toString(httpResponse.getEntity(), "utf-8"); 
        //解析json数据 
        ObjectMapper objectMapper = new ObjectMapper(); 
        VerifyBean verifyResult = objectMapper.readValue(result, VerifyBean.class); 
        
        //验证通过,应用返回浏览器需要验证的页面 
        if(verifyResult.getRet().equals("0")) { 
            Auth auth = new Auth(); 
            auth.setUserId(verifyResult.getUserId()); 
            auth.setUsername(verifyResult.getUsername()); 
            auth.setGlobalId(verifyResult.getGlobalId()); 
            request.getSession().setAttribute("auth", auth); //建立本地会话 
            return "redirect:http://" + returnURL; 
        } 
    } 
} catch (Exception e) { 
    return "redirect:" + loginURL; 
}
```

#### SSO Server接口

核心实现细节讨论清楚了，我们就可以根据登录登出操作流程，定义SSO Server和SSO Client所提供的接口。SSO Server认证中心包含4个重要接口，分别如下：

**接口一：** `/page/login`。此接口主要接受来自应用系统的认证请求，此时，returnURL参数需加上，用以向认证中心标识是哪个应用系统，以及返回该应用的URL。如用户没有登录，应用中心向浏览器用户显示登录页面。如已登录，则产生临时令牌token，并重定向回该系统。上面登录时序交互图中的2和此接口有关。当然，该接口也同时接受用户直接向认证中心登录，此时没有returnURL参数，认证中心直接返回登录页面。
```bash
 接口名称：/page/login
 入参： returnURL (系统URL，可选)
 返回： 1.显示登录页面；2.产生临时认证token并重定向回系统；
```

**接口二：** `/auth/login`。处理浏览器用户登录认证请求。如带有returnURL参数，认证通过后，将产生临时认证令牌token，并携带此token重定向回系统。如没有带returnURL参数，说明用户是直接从认证中心发起的登录请求，认证通过后，返回认证中心首页提示用户已登录。上面登录时序交互图中的3和此接口有关。
```bash
 接口名称：/auth/login
 入参： username[*用户名]、password[*密码]、returnURL
 返回： 1.产生临时认证token并重定向回系统；2.返回认证中心首页提示登录成功；
```

**接口三：** `/auth/verify`。认证应用系统来的token是否有效，如有效，应用系统向认证中心注册，同时认证中心会返回该应用系统登录用户的相关信息，如ID,username等。上面登录时序交互图中的4和此接口有关。
```bash
 接口名称：/auth/verify
 入参： token、localid
 返回： JSON格式消息
    {
        ret: 返回结果字符串，0表示成功；
        msg: 返回结果文字说明字符串；
        userid: 用户ID;
        username: 用户登录名;
        globalid: 全局会话ID,登出时使用;
    }
```
**接口四：** `/auth/logout`。登出接口处理两种情况，一是直接从认证中心登出，一是来自应用重定向的登出请求。这个根据gId来区分，无gId参数说明直接从认证中心注销，有，说明从应用中来。接口首先取消当前全局登录会话，其次根据注册的已登录应用，通知它们进行登出操作。
```bash
 接口名称：/auth/logout
 入参： gid[全局会话id,可选]
 返回： 1.返回OK; 2.返回认证中心首页;
```

#### SSO Client接口

**接口一：** `/auth/check`。接收来自认证中心携带临时令牌token的重定向，向认证中心/auth/verify接口去验证此token的有效性，如有效，即建立本地会话，根据returnURL返回浏览器用户的实际请求。如验证失败，再重定向到认证中心登录页面。
```bash
 接口名称：/auth/check
 入参： token[*登录token]
 返回： 成功重定向returnURL，失败重定向到登录页面;
```

**接口二：** `/auth/logout`。处理两种情况，一种是浏览器向本应用接口发出的直接登出请求，应用会消除本地会话，调用认证服务器/auth/logout接口，通知认证中心删除全局会话和其它已登录应用的本地会话。 如果是从认证中心来的登出请求，此时带有localId参数，接口实现会直接删除本地会话，返回字符串"ok"。
```bash
 接口名称：/auth/logout
 入参： localid[本地会话id,可选]
 返回： 1.首页; 2.'ok'字符串;
```


## CAS
CAS是中央认证服务Central Authentication Service的简称。最初由耶鲁大学的Shawn Bayern 开发，后由Jasig社区维护，经过十多年发展，目前已成为影响最大、广泛使用的、基于Java实现的、开源SSO解决方案。
2012年，Jasig和另一个有影响的组织Sakai Foundation合并，组成Apereo。Apereo是一个由高等学术教育机构发起组织的联盟，旨在为学术教育机构提供高质量软件，当然很多软件也被大量应用于商业环境，譬如CAS。目前CAS由Apereo社区维护。
CAS的官方网址是： https://www.apereo.org/projects/cas
工程代码网址：https://github.com/Jasig/cas

CAS也提供了一个认证中心，叫CAS Server，参与登录的应用系统都会引导到CAS Server进行登录。各应用系统与CAS Server交互通信的登录组件叫CAS Client。如CAS Client，已经提供了包括Java、.net、php、ruby、perl等多种语言的实现，非常适合异构系统的单点登录使用场景。再比如认证方式，除了常见的基于数据库认证，还提供LDAP使用场景,同时支持各种常见认证协议，如spnego、OpenId、X509等等。对于全局会话，CAS基于Cookie使用了自己的实现方式，而服务端的会话存储，除了缺省基于内存模式，还提供了基于ehcache、memcached等多种实现，同时提供了灵活接口便于自己定制扩展，这非常适合某些高可用性、高性能的应用场景。因此，在一般场景下，我们不需要重新发明轮子，直接在成熟技术框架基础上开发使用即可。这也是CAS在很多互联网和企业应用中广泛使用的原因。当然，对于某些场景，如安全性因素、更特殊更高效的应用场景，在技术实力许可的情况下，通常都自己实现SSO。

