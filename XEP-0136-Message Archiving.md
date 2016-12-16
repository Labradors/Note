# XEP-0136: Message Archiving

这个文档定义了XMPP服务端消息存取的机制与参数配置。

注意：本文定义的协议是XMPP标准基金会的标准草案。我们鼓励实现此协议，并且适用于生产环境。但在成为正式标准之前，可能又一些更改。我那个操。

## 介绍

注意：XMPP标准基金会认为消息归档协议过于复杂。并且可能在将其推进到Final之前请求对此规范进行彻底审查和简化。

很多XMPP客户端实现了类似客户端存档的功能。但是要信息同步，服务端的消息保存是必要的。要解决在多个端保存数据有两种方式：

- 手动归档，客户端手动发送消息到服务器进行保存。
- 自动归档，服务端接受到消息后自动保存。

这个文档定义了如何取回消息和如何管理消息。

注意：服务器必须响应type为`get`,`set`的`<iq/>`消息。

## 归档首选项

### 介绍

不是所有的用户都希望保存信息，所以，是否归档应当可配置。同时，客户端应当可根据特定联系人保存特定的信息。在客户端，无论选择哪种归档方式，都应当遵循协议规定，建议客户端保存的优先级高于服务器优先级。本章讨论以下几种用例：

- 客户端决定保存的模式为默认保存，离线保存还是特定联系人保存。
- 客户单决定保存模式为默认保存，离线保存。
- 客户端为特定用户默认保存，离线保存。
- 客户端为特定会话设置保存模式。

### 语法介绍

归档信息中有下面几个子节点，分别是`<pref/>`,`<auto/>`,`<default/>`,`<item/>`,`<method/>`下面进行详细讲解。为了获得完整的信息，当服务端返回归档消息到客户端时：

- 必须包含`<auto/>`标签，设置自动存档是否打开。
- 必须包含`<default/>`,以确定是否使用ORT模式。
- 可选的包含一个或者多个<item/>确定对特定用户的设置。
- 可选的包含一个或者多个<session/>，确定会话使用的归档设置。
- 必须至少包含三个<method/>，设置消息归档的类型。

完整的例子1:完整的配置

```xml
<iq type='result' id='pref1' to='juliet@capulet.com/chamber'>
  <pref xmlns='urn:xmpp:archive'>
    <auto save='false'/>
    <default expire='31536000' otr='concede' save='body'/>
    <item jid='romeo@montague.net' otr='require' save='false'/>
    <item expire='630720000' jid='benvolio@montague.net' otr='forbid' save='message'/>
    <session thread='ffd7076498744578d10edabfe7f4a866' save='body'/>
    <method type='auto' use='forbid'/>
    <method type='local' use='concede'/>
    <method type='manual' use='prefer'/>
  </pref>
</iq>
```

#### Auto节点

<auto />元素指定此流的当前自动归档设置。它必须包含一个属性`save`,设置存在于这个流的信息是否保存。还可以设置一个`scope`属性，他有两个值。

- global,设置下一个流的save模式跟当前相同。
- stream，save只使用于当前流。对于下一个流，单独指定。

#### Default 节点

<default />元素指定OTR模式和保存模式的默认设置。元素必须为空，并且必须包含一个`otr`属性和一个`save`属性。元素可以包括`expire`属性。

- expire  属性

如果'save'属性没有设置为`false`，推荐也包含一个`expire`属性，它表示服务器应该在多长时间内归档消息，服务器应该删除它们。

- otr元素

`otr`属性指定OTR模式的用户默认设置。允许值为：

1. approve-用户必须明确同意离线消息
2. concede-如果另一个用户请求，通信可能会关闭记录。
3. forbid--通信不能关闭记录。
4. oppose--通信不应该关闭记录，即使请求。
5. prefer--如果可能的话，应该关闭记录。
6. require--通信必须关闭记录。

- save属性

1. body，保存消息内容。
2. false，不保存
3. message，保存所有xml信息。
4. stream，保存所有流信息。

注意：在本地存档时，客户端可以保存每个<message />元素的完整XML内容，即使保存模式为“body”。

#### Item节点

<item />元素指定关于特定实体的OTR模式和保存模式的设置

他的元素必须是空的并且必须包含一个`jid`属性，一个`otr`属性和一个`save`节点，可选的包含expire节点。

- expire节点

如果`save`属性没有设置`false`，推荐也包括一个`expire`属性，它表示服务器应该在多长时间内存储消息，服务器应该删除它们。

- jid节点

`jid`属性指定在此<item />元素中指定的首选项所适用的XMPP实体的JabberID，请参阅本文档的JID匹配部分。

- `otr`属性，与上述一致。
- `save`属性，与上述一致。

#### 会话节点

必须包含一个`thread`,`save`属性。可选`timeout`属性。当流被关闭了，在服务端应当删除掉所有session节点。

- timeout，设置超时时间。
- thread，指定当前会话的线程id。
- save，同上

#### Method节点

指定归档的方法，必须指定两个属性，`type`,`use`。

### 指定配置

为了得到服务端默认的配置。发送

```xml
<iq type='get' id='pref1'>
  <pref xmlns='urn:xmpp:archive'/>
</iq>
```

 得到以下结果:

```xml
<iq type='result' id='pref1' to='juliet@capulet.com/chamber'>
  <pref xmlns='urn:xmpp:archive'>
    <auto save='false'/>
    <default expire='31536000' otr='concede' save='body'/>
    <item jid='romeo@montague.net' otr='require' save='false'/>
    <item expire='630720000' jid='benvolio@montague.net' otr='forbid' save='message'/>
    <session thread='ffd7076498744578d10edabfe7f4a866' save='body'/>
    <method type='auto' use='forbid'/>
    <method type='local' use='concede'/>
    <method type='manual' use='prefer'/>
  </pref>
</iq>
```

```xml
<iq type='result' id='pref1' to='juliet@capulet.com/chamber'>
  <pref xmlns='urn:xmpp:archive'>
    <default otr='concede' save='false' unset='true'/>
    <method type='auto' use='concede'/>
    <method type='local' use='concede'/>
    <method type='manual' use='concede'/>
    <auto save='false'/>
  </pref>
</iq>
```

### 设置默认模式

客户端设置默认模式：

```xml
<iq type='set' id='pref2'>
  <pref xmlns='urn:xmpp:archive'>
    <default otr='prefer' save='false'/>
  </pref>
</iq>
```

如果服务器响应了更改：

```xml
<iq type='result' id='pref2' to='juliet@capulet.com/chamber'/>
```

### 设置contact

更改

```xml
<iq type='set' id='pref3'>
  <pref xmlns='urn:xmpp:archive'>
    <item jid='romeo@montague.net' save='body' expire='604800' otr='concede'/>
  </pref>
</iq>
```

```xml
<iq type='result' id='pref3' to='juliet@capulet.com/chamber'/>
```

```xml
<iq type='set' id='push3' to='juliet@capulet.com/chamber'>
  <pref xmlns='urn:xmpp:archive'>
    <item jid='romeo@montague.net' save='body' expire='604800' otr='concede'/>
  </pref>
</iq>

<iq type='set' id='push4' to='juliet@capulet.com/pda'>
  <pref xmlns='urn:xmpp:archive'>
    <item jid='romeo@montague.net' save='body' expire='604800' otr='concede'/>
  </pref>
</iq>
```

移除contact更改

```xml
<iq type='set' id='remove1'>
  <itemremove xmlns='urn:xmpp:archive'>
    <item jid='benvolio@montague.net'/>
  </itemremove>
</iq>
```

服务器响应

```xml
<iq type='result' id='remove1' to='juliet@capulet.com/chamber'/>
```

###  会话更改

设置会话更改

```xml
<iq type='set' id='pref4'>
  <pref xmlns='urn:xmpp:archive'>
    <session thread='ffd7076498744578d10edabfe7f4a866' save='body' otr='concede'/>
  </pref>
</iq>
```

服务器响应

```xml
<iq type='result' id='pref4' to='juliet@capulet.com/chamber'/>
```

```xml
<iq type='set' id='push5' to='juliet@capulet.com/chamber'>
  <pref xmlns='urn:xmpp:archive'>
    <session thread='ffd7076498744578d10edabfe7f4a866' save='body' timeout='3600' otr='concede'/>
  </pref>
</iq>

<iq type='set' id='push6' to='juliet@capulet.com/pda'>
  <pref xmlns='urn:xmpp:archive'>
    <session thread='ffd7076498744578d10edabfe7f4a866' save='body' timeout='3600' otr='concede'/>
  </pref>
</iq>
    
```

移除更改

```xml
<iq type='set' id='remove2'>
  <sessionremove xmlns='urn:xmpp:archive'>
    <session thread='ffd7076498744578d10edabfe7f4a866'/>
  </sessionremove>
</iq>
```

服务器响应

```xml
<iq type='result' id='remove2' to='juliet@capulet.com/chamber'/>
```

### 设置归档方法

 设置

```xml
<iq type='set' id='pref5'>
  <pref xmlns='urn:xmpp:archive'>
    <method type='auto' use='concede'/>
    <method type='local' use='forbid'/>
    <method type='manual' use='prefer'/>
  </pref>
</iq>
```

服务器响应

```xml
<iq type='result' id='pref5' to='juliet@capulet.com/chamber'/>
```

```xml
<iq type='set' id='push7' to='juliet@capulet.com/chamber'>
  <pref xmlns='urn:xmpp:archive'>
    <method type='auto' use='concede'/>
    <method type='local' use='forbid'/>
    <method type='manual' use='prefer'/>
  </pref>
</iq>

<iq type='set' id='push8' to='juliet@capulet.com/pda'>
  <pref xmlns='urn:xmpp:archive'>
    <method type='auto' use='concede'/>
    <method type='local' use='forbid'/>
    <method type='manual' use='prefer'/>
  </pref>
</iq>
    
```

## 离线记录

暂时不管，重心不在这

## 集合：Chat节点

## 手动归档

## 自动归档

## 归档管理

重点，我想看的就是这儿。归档管理有以下三种方式：

1. 检索集合列表
2. 检索一个集合
3. 移除一个集合

### 检索集合列表

- 请求具有相同JID的列表的第一页

```xml
<iq type='get' id='juliet1'>
  <list xmlns='urn:xmpp:archive'
        with='juliet@capulet.com'>
    <set xmlns='http://jabber.org/protocol/rsm'>
      <max>30</max>
    </set>
  </list>
</iq>
```

- 请求具有相同JID，在两个时间之内的第一页

```xml
<iq type='get' id='period1'>
  <list xmlns='urn:xmpp:archive'
        with='juliet@capulet.com'
        start='1469-07-21T02:00:00Z'
        end='1479-07-21T04:00:00Z'>
    <set xmlns='http://jabber.org/protocol/rsm'>
      <max>30</max>
    </set>
  </list>
</iq>
    
```

- 请求在一个时间之后的第一页

```xml
<iq type='get' id='list1'>
  <list xmlns='urn:xmpp:archive'
        start='1469-07-21T02:00:00Z'>
    <set xmlns='http://jabber.org/protocol/rsm'>
      <max>30</max>
    </set>
  </list>
</iq>
```

- 取回一个空的列表

```xml
<iq type='result' to='romeo@montague.net/orchard' id='list1'>
  <list xmlns='urn:xmpp:archive'/>
</iq>
```

- 取回第二页

```xml
<iq type='get' id='list2'>
  <list xmlns='urn:xmpp:archive'
        start='1469-07-21T02:00:00Z'>
    <set xmlns='http://jabber.org/protocol/rsm'>
      <max>30</max>
      <after>1469-07-21T03:16:37Zbalcony@house.capulet.com</after>
    </set>
  </list>
</iq>
```