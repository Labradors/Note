# 配置运营商Sip网关


上一篇文章记录了FreeSwitch创建新用户，流程大概是这样，先拿到运营商的sip账号，配置网关，然后重载配置，在呼出计划中添加正则表达式，让系统知道，系统号码走系统网关，运营商的号码走运营商的网关。
<!--more-->

## 网关配置

- 创建配置文件

```she
cd /etc/freeswitch/sip_profiles/external
vim gw1.xml
```

- 配置

```xml
<include>
  <gateway name="gw1">
  <!--/// account username *required* ///-->
  <param name="username" value="5244"/>
  <!--/// auth realm: *optional* same as gateway name, if blank ///-->
  <param name="realm" value="39.108.168.70"/>
  <!--/// username to use in from: *optional* same as  username, if blank ///-->
  <!--<param name="from-user" value="cluecon"/>-->
  <!--/// domain to use in from: *optional* same as  realm, if blank ///-->
  <!--<param name="from-domain" value="asterlink.com"/>-->
  <!--/// account password *required* ///-->
  <param name="password" value="UwEG7APd"/>
  <!--/// extension for inbound calls: *optional* same as username, if blank ///-->
  <!--<param name="extension" value="cluecon"/>-->
  <!--/// proxy host: *optional* same as realm, if blank ///-->
  <!--<param name="proxy" value="asterlink.com"/>-->
  <!--/// send register to this proxy: *optional* same as proxy, if blank ///-->
  <!--<param name="register-proxy" value="mysbc.com"/>-->
  <!--/// expire in seconds: *optional* 3600, if blank ///-->
  <!--<param name="expire-seconds" value="60"/>-->
  <!--/// do not register ///-->
  <!--<param name="register" value="false"/>-->
  <!-- which transport to use for register -->
  <!--<param name="register-transport" value="udp"/>-->
  <!--How many seconds before a retry when a failure or timeout occurs -->
  <param name="retry-seconds" value="30"/>
  <!--Use the callerid of an inbound call in the from field on outbound calls via this gateway -->
  <!--<param name="caller-id-in-from" value="false"/>-->
  <!--extra sip params to send in the contact-->
  <!--<param name="contact-params" value=""/>-->
  <!-- Put the extension in the contact -->
  <!--<param name="extension-in-contact" value="true"/>-->
  <!--send an options ping every x seconds, failure will unregister and/or mark it down-->
  <param name="ping" value="25"/>
  <!--<param name="cid-type" value="rpid"/>-->
  <!--rfc5626 : Abilitazione rfc5626 ///-->
  <!--<param name="rfc-5626" value="true"/>-->
  <!--rfc5626 : extra sip params to send in the contact-->
  <!--<param name="reg-id" value="1"/>-->
  </gateway>
</include>
```

- 重新加载配置文件

```shell
sofia profile external rescan
```

- 查看注册状态

```shell
sofia status 
```

## 分机呼出配置

- 创建配置文件

```shell
/etc/freeswitch/dialplan/default
vim call_out.xml
```

- 配置

```xml
<include>
<extension name="call out">
<condition field="destination_number" expression="^0(\d+)$">
<action application="bridge" data="sofia/gateway/gw1/$1"/>
</condition>
</extension>
</include>
```

- 重载

```shell
reloadxml
```

完工。