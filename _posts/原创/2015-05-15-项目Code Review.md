---
layout: post
title:  "项目Code Review"
date:   2017-06-01 11:03:01 +0800
categories: 原创
tag: spring 原创
sid: 1496316757
---

> 最近Code Review自己以及同事的业务代码,发现存在的问题非常之多,这里以阿里公开的手册《阿里巴巴Java开发手册》为对比

## 一、编程规范
### (一)、命名规约
1. 【强制】 代码中的命名均不能以下划线或美元符号开始，也不能以下划线或美元符号结束。
反例： _name / __name / $Object / name_ / name$ / Object$

    **代码尚未出现类似命名变量**

2. 【强制】 代码中的命名严禁使用拼音与英文混合的方式，更不允许直接使用中文的方式。
说明： 正确的英文拼写和语法可以让阅读者易于理解，避免歧义。注意，即使纯拼音命名方式
也要避免采用。
反例： DaZhePromotion [打折] / getPingfenByName() [评分] / int 某变量 = 3
正例： alibaba / taobao / youku / hangzhou 等国际通用的名称， 可视同英文。

    **公司反例**

    ~~~java
        //    common普通消息0   系统消息（摄影社区账号）1
       //    like(点赞):2   follow(关注)3
       //    comment(评论)4  zuopin(作品) 5
       //    buluo(部落)6   qianyue(签约)7
       //    huodong(活动)8
         public enum IdType{
               qianyue,
               buluo,
               zuopin,
               huodong,
               shequ,
               like,
               follow,
               comment
           }
        private  String qianyuexiaoxi  = "41eb2d310452fb74232e08954dbbf3712";

           private  String buluoxiaoxi = "79dc0fc6144978de340d7f06531d91200";

           private  String zuopinxiaoxi = "78bbb7f644b7981d0c0e93dcceb364091";

           private  String huodongxiaoxi = "4517a02bf40fbadc1b4e0ba8763504335";

           private  String sheyingshequ  = "cf33791224fa993890f992da405e47519" ;

           private  String shequlike  = "e6013139b4ffb84f9ac65c49da9a17086" ;

           private  String shequfollow  = "201eac260445f93daed17ca339dcd1208" ;

           private  String shequcomment  = "2d0730c84458d9bb4ada193ceb4f85496" ;
       ~~~
    说明:成员变量中大量的使用了拼音的方式,且与英文拼写混淆一起

3. 【强制】类名使用 UpperCamelCase 风格，必须遵从驼峰形式，但以下情形例外： （ 领域模型
的相关命名） DO / BO / DTO / VO 等。
正例： MarcoPolo / UserDO / XmlService / TcpUdpDeal / TaPromotion
反例： macroPolo / UserDo / XMLService / TCPUDPDeal / TAPromotion

    **公司反例**

    ~~~java
    public class ApkAction
    ~~~
    说明:正确的命名为APKAction,其中APK属于一个缩写名词

4. 【强制】方法名、参数名、成员变量、局部变量都统一使用 lowerCamelCase 风格，必须遵从
驼峰形式。
正例： localValue / getHttpMessage() / inputUserId

    **公司反例**

    ~~~java
    public String about(String userId, String queriedUserId, HttpServletRequest request, String type,
                            String imgsize, String avatarsize, int page, int size) {
    ~~~
    说明:参数中的imgsize,avatarsize应该为imgSize,avatarSize

5. 【 强制】常量命名全部大写，单词间用下划线隔开，力求语义表达完整清楚，不要嫌名字长。
正例： MAX_STOCK_COUNT
反例： MAX_COUNT

    **公司反例**

    ~~~java
    private final static String idKey = "PF500_ids";
        private final static String idHashKey = "PF500_hash_ids";
    ~~~
    说明:常量定义成驼峰式,应该为全部大写,切以下划线隔开,如ID_KEY,ID_HASHKEY,且该命名并无法完整表达

6. 【强制】抽象类命名使用 Abstract 或 Base 开头； 异常类命名使用 Exception 结尾； 测试类
命名以它要测试的类的名称开始，以 Test 结尾。

    **公司反例**

    ~~~java
    public abstract class MainMsgHandler
    ~~~
    说明:无法从命名上体现出该类可能是个抽象类

7. 【强制】中括号是数组类型的一部分，数组定义如下： String[] args;
反例： 请勿使用 String args[]的方式来定义。

    **公司反例**

    ~~~java
    String tmp[] = fileName.split("\\.");
            Object arg = targetObj ;
            for (int i = 0; i < tmp.length; i++) {
                Method methdo = arg.getClass().
                        getMethod(getGetterNameByFiledName(tmp[i]));
                arg = methdo.invoke(arg);
            }
            return arg ;
    ~~~
说明:本以为不会存在此类代码,居然真有
8. 【强制】 POJO 类中布尔类型的变量，都不要加 is，否则部分框架解析会引起序列化错误。
反例： 定义为基本数据类型 boolean isSuccess； 的属性，它的方法也是 isSuccess()， RPC
框架在反向解析的时候， “以为”对应的属性名称是 success，导致属性获取不到，进而抛出异
常。

   **公司反例**

    ~~~java
     protected boolean isSynchro = true;
      protected static boolean isDebug = false;//用于调试代码
    ~~~

   说明:POJO 类中布尔类型的变量，都不要加 is

9. 【强制】包名统一使用小写，点分隔符之间有且仅有一个自然语义的英语单词。包名统一使用
单数形式，但是类名如果有复数含义，类名可以使用复数形式。
正例： 应用工具类包名为 com.alibaba.open.util、类名为 MessageUtils（ 此规则参考
spring 的框架结构）
    **公司反例**

    说明:除了包名,工具类这个并无强制要求

10. 【强制】杜绝完全不规范的缩写， 避免望文不知义。
反例： AbstractClass“ 缩写” 命名成 AbsClass； condition“ 缩写” 命名成 condi，此类
随意缩写严重降低了代码的可阅读性。

    **公司反例**
    说明:暂无

11. 【推荐】如果使用到了设计模式，建议在类名中体现出具体模式。
说明： 将设计模式体现在名字中，有利于阅读者快速理解架构设计思想。
正例： public class OrderFactory;
public class LoginProxy;
public class ResourceObserver;
    **公司反例**
    说明:暂无

12. 【推荐】接口类中的方法和属性不要加任何修饰符号（ public 也不要加） ，保持代码的简洁
性，并加上有效的 Javadoc 注释。尽量不要在接口里定义变量，如果一定要定义变量，肯定是
与接口方法相关，并且是整个应用的基础常量。
正例： 接口方法签名： void f();
接口基础常量表示： String COMPANY = "alibaba";
反例： 接口方法定义： public abstract void f();
说明： JDK8 中接口允许有默认实现，那么这个 default 方法，是对所有实现类都有价值的默
认实现。

    **公司反例**

    说明:基本所有公司不一定遵循该约定

13. 接口和实现类的命名有两套规则：
1） 【强制】对于 Service 和 DAO 类，基于 SOA 的理念，暴露出来的服务一定是接口，内部
的实现类用 Impl 的后缀与接口区别。
正例： CacheServiceImpl 实现 CacheService 接口。
2）【推荐】 如果是形容能力的接口名称，取对应的形容词做接口名 （ 通常是–able 的形式）。
正例： AbstractTranslator 实现 Translatable。
说明:基本所有公司都遵循该约定
14. 【参考】枚举类名建议带上 Enum 后缀，枚举成员名称需要全大写，单词间用下划线隔开。
说明： 枚举其实就是特殊的常量类，且构造方法被默认强制是私有。
正例： 枚举名字： DealStatusEnum， 成员名称： SUCCESS / UNKOWN_REASON。
    **公司反例**
    ~~~java
    public enum  ContestSearchType
    public enum UserRole
    public enum TribeUserState
    ~~~

    说明:应该在类名称后面加上Enum

15. 【参考】各层命名规约：
A) Service/DAO 层方法命名规约
1） 获取单个对象的方法用 get 做前缀。
2） 获取多个对象的方法用 list 做前缀。
3） 获取统计值的方法用 count 做前缀。
4） 插入的方法用 save（ 推荐） 或 insert 做前缀。
5） 删除的方法用 remove（ 推荐） 或 delete 做前缀。
6） 修改的方法用 update 做前缀。
B) 领域模型命名规约
1） 数据对象： xxxDO， xxx 即为数据表名。
2） 数据传输对象： xxxDTO， xxx 为业务领域相关的名称。
3） 展示对象： xxxVO， xxx 一般为网页名称。
4） POJO 是 DO/DTO/BO/VO 的统称，禁止命名成 xxxPOJO。

    **这个不同公司的确有不同的规范**

### (二)、常量定义

1. 【强制】不允许出现任何魔法值（ 即未经定义的常量） 直接出现在代码中。
反例： String key="Id#taobao_"+tradeId；
cache.put(key, value);
说明：该出的举例并不明确,请参考:[http://blog.csdn.net/mengxianhua/article/details/7402572](http://blog.csdn.net/mengxianhua/article/details/7402572)
2. 【强制】 long 或者 Long 初始赋值时，必须使用大写的 L，不能是小写的 l，小写容易跟数字
1 混淆，造成误解。
说明： Long a = 2l; 写的是数字的 21，还是 Long 型的 2?

    **公司反例**
    ~~~java
    long time = System.currentTimeMillis() - 86400000l;
    ~~~
    说明：86400000l容易被看错864000001,这是大忌

3. 【推荐】不要使用一个常量类维护所有常量，应该按常量功能进行归类，分开维护。如：缓存
相关的常量放在类： CacheConsts 下； 系统配置相关的常量放在类： ConfigConsts 下。
说明： 大而全的常量类，非得使用查找功能才能定位到修改的常量，不利于理解和维护。

    **公司反例**

    说明：暂无

4. 【推荐】常量的复用层次有五层：跨应用共享常量、应用内共享常量、子工程内共享常量、包
内共享常量、类内共享常量。
    - 1） 跨应用共享常量：放置在二方库中，通常是 client.jar 中的 constant 目录下。
    - 2） 应用内共享常量：放置在一方库的 modules 中的 constant 目录下。
反例： 易懂变量也要统一定义成应用内共享常量，两位攻城师在两个类中分别定义 了
表示“是”的变量：
类 A 中： public static final String YES = "yes";
类 B 中： public static final String YES = "y";
A.YES.equals(B.YES)，预期是 true，但实际返回为 false，导致产生线上问题。
    - 3） 子工程内部共享常量：即在当前子工程的 constant 目录下。
    - 4） 包内共享常量：即在当前包下单独的 constant 目录下。
    - 5） 类内共享常量：直接在类内部 private static final 定义。

### (三)、OOP规约

1. 【强制】避免通过一个类的对象引用访问此类的静态变量或静态方法，无谓增加编译器解析成
本，直接用类名来访问即可。

    **公司反例**
    ~~~java
    //定义
    public static JSONObject fixPictureListData(JSONObject data){
            if (data==null) return data;
            JSONArray jsonArray = new JSONArray();
            jsonArray.add(data);
            return fixPictureListData(jsonArray).getJSONObject(0);
        }
        //调用
         info = cHub.fixPictureListData(info);
         temp = resHub.fixPictureListData(temp, new String[]{ "picturePvCount", "categoryId"});
    ~~~
    说明：无谓增加编译器解析成
       本

2. 【强制】所有的覆写方法，必须加@Override 注解。
反例： getObject()与 get0bject()的问题。一个是字母的 O，一个是数字的 0，加@Override
可以准确判断是否覆盖成功。另外，如果在抽象类中对方法签名进行修改，其实现类会马上编
译报错。

    说明：这个一般由工具生成

3. 【强制】相同参数类型，相同业务含义，才可以使用 Java 的可变参数，避免使用 Object。
说明： 可变参数必须放置在参数列表的最后。 （ 提倡同学们尽量不用可变参数编程）
正例： public User getUsers(String type, Integer... ids)

    说明：虽然手册上说明提倡同学们尽量不用可变参数编程,但是阿里每个开源项目都大量使用了可变参数


4. 【强制】对外暴露的接口签名，原则上不允许修改方法签名，避免对接口调用方产生影响。接
口过时必须加@Deprecated 注解，并清晰地说明采用的新接口或者新服务是什么。

    说明:总感觉@Deprecated很鸡肋,跟JDK-API一样,虽然标注了,但是限于环境,还是必须使用

5. 【强制】不能使用过时的类或方法。
说明： java.net.URLDecoder 中的方法 decode(String encodeStr) 这个方法已经过时，应
该使用双参数 decode(String source, String encode)。接口提供方既然明确是过时接口，
那么有义务同时提供新的接口； 作为调用方来说，有义务去考证过时方法的新实现是什么。

    说明:总感觉@Deprecated很鸡肋,跟JDK-API一样,虽然标注了,但是限于环境,还是必须使用

6. 【强制】 Object 的 equals 方法容易抛空指针异常，应使用常量或确定有值的对象来调用
equals。
正例： "test".equals(object);
反例： object.equals("test");
说明： 推荐使用 java.util.Objects#equals （ JDK7 引入的工具类）

    **公司反例**
     ~~~java
     if (!type.equals("json")) {
                 return "contest/index";
          }
              if (listType.equals("3")) {
     ~~~

    说明：公司大量使用了object.equals("test");类似的方式,很不好

### (七)、控制语句

1. 【强制】在一个 switch 块内，每个 case 要么通过 break/return 等来终止，要么注释说明程
序将继续执行到哪一个 case 为止； 在一个 switch 块内，都必须包含一个 default 语句并且
放在最后，即使它什么代码也没有。

    说明:公司除了我,好像没人喜欢用switch,都是长条的if ...else

2. 【强制】在 if/else/for/while/do 语句中必须使用大括号，即使只有一行代码，避免使用
下面的形式： if (condition) statements;

    **公司反例**
    ~~~java
    Resource photo = contestHub.getObject(Resource.class , resourceId);
    if (photo==null) throw new Exception("作品不存在");
    User user = contestHub.getObject(User.class , userId);
    if (user==null) throw new Exception("用户不存在");
    if (contestCategories == null) contestCategories = new ArrayList<ContestCategory>();
    ~~~

    说明:不应该写成行内格式

3. 【推荐】推荐尽量少用 else， if-else 的方式可以改写成：
if(condition){
...
return obj;
}
// 接着写 else 的业务逻辑代码;
说明： 如果非得使用 if()...else if()...else...方式表达逻辑，【强制】请勿超过 3 层，
超过请使用状态设计模式。

     **公司反例**
     ~~~java
     if (searchType.equals("createTimeDesc")) {
         sql.append(" and createdTime > "+begin+" and createdTime < "+end+" order by createdTime desc");
     } else if (searchType.equals("createTimeAsc")){
         sql.append(" and createdTime > "+begin+" and createdTime < "+end+" order by createdTime ");
     } else if (searchType.equals("dateTimeDesc")){
         sql.append(" and exifInfo.dateTimeDigitized > "+begin+" and exifInfo.uploadTime < "+end+" order by exifInfo.uploadTime  desc");
     } else if (searchType.equals("dateTimeAsc")){
         sql.append(" and exifInfo.dateTimeDigitized > "+begin+" and exifInfo.uploadTime < "+end+" order by  exifInfo.uploadTime ");
     }
     ~~~

    ~~~java
        else {
        if(pushName.equals("comment")){
            pushSet.setComment(pushValue);
        }else if(pushName.equals("praise")){
            pushSet.setPraise(pushValue);
        }else if(pushName.equals("follow")){
            pushSet.setFollow(pushValue);
        }else if(pushName.equals("resourceMessage")){
            pushSet.setResourceMessage(pushValue);
        }else if(pushName.equals("tribeMessage")){
            pushSet.setTribeMessage(pushValue);
        }else if(pushName.equals("contractMessage")){
            pushSet.setContractMessage(pushValue);
        }else if(pushName.equals("activityMessage")){
            pushSet.setActivityMessage(pushValue);
        }else if(pushName.equals("mail")){
            pushSet.setMail(pushValue);
        }
        mysqlService.update(pushSet);
        }
    ~~~

    说明：公司中大量使用了类似的语法

    正例： 逻辑上超过 3 层的 if-else 代码可以使用卫语句，或者状态模式来实现。

4. 【推荐】除常用方法（如 getXxx/isXxx）等外，不要在条件判断中执行其它复杂的语句，将复
杂逻辑判断的结果赋值给一个有意义的布尔变量名，以提高可读性。
说明： 很多 if 语句内的逻辑相当复杂，阅读者需要分析条件表达式的最终结果，才能明确什么
样的条件执行什么样的语句，那么，如果阅读者分析逻辑表达式错误呢？
正例：
//伪代码如下
boolean existed = (file.open(fileName, "w") != null) && (...) || (...);
if (existed) {
...
}
反例：
if ((file.open(fileName, "w") != null) && (...) || (...)) {
...
}

    **公司反例**
    ~~~java
    if (!"contest".equals(contestCategory) && !"activity".equals(contestCategory) &&!"collection".equals(contestCategory) ){
    ~~~

    说明:并不知道该判断条件的目的

5. 【推荐】循环体中的语句要考量性能，以下操作尽量移至循环体外处理，如定义对象、变量、
获取数据库连接，进行不必要的 try-catch 操作（ 这个 try-catch 是否可以移至循环体外） 。

    **公司反例**
    ~~~java
    try {
                String objNameF = ClassUtil.getKeyPrefix(linkEnum.fromType().objectClass().getName());
                String objNameT = ClassUtil.getKeyPrefix(linkEnum.toType().objectClass().getName());
                String redisKey = linkEnum.fromKey().redisKey();
                String setKey = objNameF+":"+fromId+":"+objNameT+":"+redisKey;
                RedisService redisService =  (RedisService)SpringUtils.getBean("redisService");
                List<String> fromsort = new ArrayList<String>();
                List<String> tosort = new ArrayList<String>();
                final Map<String , Double> map = new HashMap<String, Double>();
                for(int i=0 ;i<sortList.size();i++){
                    String id = sortList.get(i);
                    Double scoreVal = redisService.opsZsetScore(setKey , id);
                    if(scoreVal!=null && scoreVal>0){
                        tosort.add(id);
                        fromsort.add(id);
                        map.put(id , scoreVal);
                    }
                }
                Collections.sort(fromsort, new Comparator<String>() {
                    public int compare(String o1, String o2) {
                        return map.get(o1).compareTo(map.get(o2));
                    }
                });
                for(int i=0;i<tosort.size() ;i++){
                    String idF = fromsort.get(i);
                    String idT = tosort.get(i);
                    if(idF.equals(idT)){
                        continue;
                    }
                    redisService.opsZsetAdd(setKey , idT , map.get(idF));
                }
            } catch (Exception e){
                e.printStackTrace();
            }
    ~~~

    说明:这段代码try的让人心碎,不可能发生异常的代码库,循环代码块

6. 【推荐】接口入参保护，这种场景常见的是用于做批量操作的接口。
7. 【参考】方法中需要进行参数校验的场景：
1） 调用频次低的方法。
2） 执行时间开销很大的方法，参数校验时间几乎可以忽略不计，但如果因为参数错误导致
中间执行回退，或者错误，那得不偿失。
3） 需要极高稳定性和可用性的方法。
4） 对外提供的开放接口，不管是 RPC/API/HTTP 接口。
5） 敏感权限入口。
8. 【 参考】方法中不需要参数校验的场景：
1） 极有可能被循环调用的方法，不建议对参数进行校验。但在方法说明里必须注明外部参
数检查。
2） 底层的方法调用频度都比较高，一般不校验。毕竟是像纯净水过滤的最后一道，参数错
误不太可能到底层才会暴露问题。一般 DAO 层与 Service 层都在同一个应用中，部署在同一
台服务器中，所以 DAO 的参数校验，可以省略。
3） 被声明成 private 只会被自己代码所调用的方法，如果能够确定调用方法的代码传入参
数已经做过检查或者肯定不会有问题，此时可以不校验参数

### (八)、注释规约

1. 【强制】 类、类属性、类方法的注释必须使用 Javadoc 规范，使用/**内容*/格式，不得使用
//xxx 方式。
说明： 在 IDE 编辑窗口中， Javadoc 方式会提示相关注释，生成 Javadoc 可以正确输出相应注
释； 在 IDE 中，工程调用方法时，不进入方法即可悬浮提示方法、参数、返回值的意义，提高
阅读效率。

    **公司反例**
    ~~~java
    /**
     * Created by DELL on 2017/2/15.
     */
    ~~~

    说明:只想说,这种开发人员,还有必要留在公司吗

2. 【强制】所有的抽象方法（ 包括接口中的方法） 必须要用 Javadoc 注释、除了返回值、参数、
异常说明外，还必须指出该方法做什么事情，实现什么功能。
说明： 对子类的实现要求，或者调用注意事项，请一并说明。

   说明：公司目前狗屎一样,提了很多次,并没有愿意多写,都是口口相述

3. 【强制】所有的类都必须添加创建者信息。
4. 【强制】方法内部单行注释，在被注释语句上方另起一行，使用//注释。方法内部多行注释
使用/* */注释，注意与代码对齐。
5. 【强制】所有的枚举类型字段必须要有注释，说明每个数据项的用途。
6. 【推荐】与其“半吊子”英文来注释，不如用中文注释把问题说清楚。专有名词与关键字保持
英文原文即可。
反例： “TCP 连接超时”解释成“传输控制协议连接超时”，理解反而费脑筋。
7. 【推荐】代码修改的同时，注释也要进行相应的修改，尤其是参数、返回值、异常、核心逻辑
等的修改。
说明： 代码与注释更新不同步，就像路网与导航软件更新不同步一样，如果导航软件严重滞后，
就失去了导航的意义。
8. 【参考】注释掉的代码尽量要配合说明，而不是简单的注释掉。
说明： 代码被注释掉有两种可能性： 1） 后续会恢复此段代码逻辑。 2） 永久不用。前者如果没
有备注信息，难以知晓注释动机。后者建议直接删掉（ 代码仓库保存了历史代码） 。
9. 【参考】对于注释的要求：第一、能够准确反应设计思想和代码逻辑； 第二、能够描述业务含
义，使别的程序员能够迅速了解到代码背后的信息。完全没有注释的大段代码对于阅读者形同
天书，注释是给自己看的，即使隔很长时间，也能清晰理解当时的思路； 注释也是给继任者看
的，使其能够快速接替自己的工作。
10. 【参考】好的命名、代码结构是自解释的，注释力求精简准确、表达到位。避免出现注释的
一个极端：过多过滥的注释，代码的逻辑一旦修改，修改注释是相当大的负担。
反例：
// put elephant into fridge
put(elephant, fridge);
方法名 put，加上两个有意义的变量名 elephant 和 fridge，已经说明了这是在干什么，语
义清晰的代码不需要额外的注释。
11. 【参考】特殊注释标记，请注明标记人与标记时间。注意及时处理这些标记，通过标记扫描，
经常清理此类标记。线上故障有时候就是来源于这些标记处的代码。
1） 待办事宜（ TODO） :（标记人，标记时间， [预计处理时间]）
表示需要实现，但目前还未实现的功能。这实际上是一个 Javadoc 的标签，目前的 Javadoc
还没有实现，但已经被广泛使用。只能应用于类，接口和方法（ 因为它是一个 Javadoc 标签） 。
2） 错误，不能工作（ FIXME） :（ 标记人，标记时间， [预计处理时间]）
在注释中用 FIXME 标记某代码是错误的，而且不能工作，需要及时纠正的情况。

    **总结:公司基本没有什么注释**

# 核心开发总结

1. 变量命名一定要规范

    变量命名一定要清晰和明确,千万不要嫌长

    我们公司中大量存在不规范的变量命名 ,比如:
     ~~~java
     JSONObject jsonObject = new JSONObject();
    String sta = PF500MClientVersion.substring(8 , PF500MClientVersion.length()-1);
    Map<Object , Object> map = redisService.opsHashEntries(SysRedisKeys.categoryHashKey);
    Integer id = -1 ;id = Integer.valueOf(catId);
    JSONObject msg = new JSONObject();
    JSONArray list = getIds(setIds , Arrays.asList(1,9,10));
    int j = contestHub.setContestById(contestId);
    .....
    ~~~
    太多类似的命名,极度不规范,让人完全不命名该变量的意义

2. 一定要检查NPE

    - 对于从其他方法或内部接口或从RPC或从HHTP或者第三方API返回的接口结果,我们一定要检查返回的结果会不会出现NullPointException
    - 对于方法连续调用的时候,一定要考虑是否在调用的过程中出现NullPointException

    我们公司中大量存在不检查NPE情况,比如:

    并没有判断contestHub.getContestBean()的返回值是否为null
    ~~~java
    ContestHub contestHub = new ContestHub();
    Integer state = contestHub.getContestBean().getInteger("state");
    ~~~

    并没有判断user是否为null
    ~~~java
    User user = resourceHub.getObject(User.class, draftPictureFinish.getString("userId"));
    resource.setId(draftPictureFinish.getString("id"));
    resource.setResourceId(draftPictureFinish.getString("id"));
    resource.setCategoryId(jsonObject.getString("categoryId"));
    resource.setTitle(draftPictureFinish.getString("title"));
    resource.setTags(draftPictureFinish.getString("keywords"));
    resource.setCreatedDate(jsonObject.getDate("date"));
    resource.setCreatedTime(jsonObject.getDate("date"));
    resource.setDescription(draftPictureFinish.getString("description"));
    resource.setUploadedDate(draftPictureFinish.getDate("createdTime"));
    resource.setHeight(draftPictureFinish.getIntValue("height"));
    resource.setWidth(draftPictureFinish.getIntValue("width"));
    resource.setLatitude(-1d);
    resource.setLongitude(-1d);
    resource.setUploaderId(user.getId());
    resource.setUploaderName(user.getNickName());
    ~~~

    并没有判断user是否为null
    ~~~java
    String PF500MClient = request.getHeader("PF500MClient");
    String userId = SSOUtils.getUserId(request);
    User user = resourceHub.getObject(User.class, userId);
    String userName = user.getNickName();
    String graphicId = UUIDUtil.generateUUID();
    ~~~

    并没有判断groupPic是否null
    ~~~java
     Resource groupPic = JSON.toJavaObject(draftBoxJson, Resource.class);
    groupPic.setId(groupPhotoId);
    groupPic.setUploaderId(userId);
    groupPic.setUploaderName(userName);
    groupPic.setResourceType(PicTypeConstant.groupPhoto);
    groupPic.setCreatedTime(new Date());
    groupPic.setCreatedDate(new Date());
    groupPic.setUploadedDate(new Date());
    ~~~

    并没有判断resource是否为null,这段代码很搞笑,其实开发已经知道需要判断,但是判断的晚了
    ~~~java
    Resource resource = userHub.getObject(Resource.class, photoId);
    String originOpenState = resource.getOpenState();
    if (resource == null) {
        throw new Exception("can not find photo and photoId : " + photoId);
    }
    if (!SSOUtils.getUserId(request).equals(resource.getUploaderId())) {
        throw new Exception("this photo is not yours and photoId" + photoId);
    }
    ~~~

3. 代码复用

    能复用的代码一定要尽量复用

    我们公司中大量存在代码冗余的代码块

    ~~~java
     private static String App_Key = SpringUtils.getBean(LoginConfig.class).getWb_app_key();
    private static String App_Secret = SpringUtils.getBean(LoginConfig.class).getWb_app_secret();
    private static String authorize_url = SpringUtils.getBean(LoginConfig.class).getWb_authorize_url();
    private static String access_token_url = SpringUtils.getBean(LoginConfig.class).getWb_access_token_url();
    private static String get_user_info = SpringUtils.getBean(LoginConfig.class).getWb_get_user_info();
    private static String WX_App_Key = SpringUtils.getBean(LoginConfig.class).getWx_app_key();
    private static String WX_App_Secret = SpringUtils.getBean(LoginConfig.class).getWx_app_secret();
    private static String WX_authorize_url = SpringUtils.getBean(LoginConfig.class).getWx_authorize_url();
    private static String WX_access_token_url = SpringUtils.getBean(LoginConfig.class).getWx_access_token_url();
    private static String WX_get_user_info = SpringUtils.getBean(LoginConfig.class).getWx_get_user_info();
    private static String GZH_App_Key = SpringUtils.getBean(LoginConfig.class).getGzh_app_key();
    private static String GZH_App_Secret = SpringUtils.getBean(LoginConfig.class).getGzh_app_secret();
    private static String GZH_authorize_url = SpringUtils.getBean(LoginConfig.class).getGzh_authorize_url();
    ~~~

    ~~~java
    JSONObject jsonObject = new JSONObject();
    jsonObject.put("status", "500");
    jsonObject.put("message", "erro");

     jsonObject.put("status","501");
    jsonObject.put("message", "不能关注自己");

     jsonObject.put("status","503");
    jsonObject.put("message", "用户已被举报");

    jsonObject.put("status","504");
    jsonObject.put("message", "用户已被删除");
    ~~~

    未完待续......