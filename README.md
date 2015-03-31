#简介
XEP-0045所实现的是传统的聊天室协议，该协议在移动终端存在诸多不便，基本上无法满足我们UUChat的群组聊天需求，因此我们需要实现一套类似精简的更能适应移动客户端的群组聊天协议扩展协议。
#范围
本文档仅讨论与群聊相关的内容，包括群组服务的发现，群组的创建，群组的删除，群组成员的添加，以及群组成员的权限和功能定义，以及群组消息的路由。
#术语
#实体用例
##发现群聊服务
* 请求发现服务
```xml
<iq from="hag66@shakespeare.lit/pda"
    id="h7ns81g"
    to="shakespeare.lit"
    type="get">
    <query xmlns="http://jabber.org/protocol/disco#items"/>
</iq>
```
* 返回已存在的服务
```xml
<iq from="shakespeare.lit"
    id="h7ns81g"
    to="hag66@shakespeare.lit/pda"
    type="result">
    <query xmlns="http://jabber.org/protocol/disco#items">
        <item jid="groupchat.shakespeare.lit"
              name="Group Chat Service"/>
    </query>
</iq>
```

##查询群聊服务支持的功能
* 查询群服务组件的功能特性
```xml
<iq from="hag66@shakespeare.lit/pda"
    id="lx09df27"
    to="groupchat.shakespeare.lit"
    type="get">
    <query xmlns="http://jabber.org/protocol/disco#info"/>
</iq>
```
* 返回群服务组件的功能特性
```xml
<iq from="groupchat.shakespeare.lit"
    id="lx09df27"
    to="hag66@shakespeare.lit/pda"
    type="result">
    <query xmlns="http://jabber.org/protocol/disco#info">
        <identity
            category="groupchat"
            name="Group Chat Service"
            type="text"/>
        <feature var="http://jabber.org/protocol/groupchat"/>
    </query>
</iq>
```

##查询群组
* 获取与用户相关的所有群组信息
```xml
<iq from="hag66@shakespeare.lit/pda"
    id="lx09df27"
    to="groupchat.shakespeare.lit"
    type="get">
    <query xmlns="http://jabber.org/protocol/groupchat#items"/>
</iq>
```
* 返回与用户相关的群组信息
```xml
<iq from="groupchat.shakespeare.lit"
    id="lx09df27"
    to="hag66@shakespeare.lit/pda"
    type="result">
    <query xmlns="http://jabber.org/protocol/groupchat#items">
        <item id="groupid1" name="Group Name 1" members="100" role="owner" />
        <item id="groupid2" name="Group Name 2" members="100" role="member" />
        <item id="groupid3" name="Group Name 3" members="100" role="member" />
    </query>
</iq>
```

##查询群组信息
* 查询指定群组的信息
```xml
<iq from="hag66@shakespeare.lit/pda"
    id="ik3vs715"
    to="groupid@groupchat.shakespeare.lit"
    type="get">
    <query xmlns="http://jabber.org/protocol/groupchat#groupinfo"/>
</iq>
```
* 返回指定群组的信息
```xml
<iq from="groupid@groupchat.shakespeare.lit"
    id="ik3vs715"
    to="hag66@shakespeare.lit/pda"
    type="result">
    <query xmlns="http://jabber.org/protocol/groupchat#groupinfo">
        <item jid="groupid@groupchat.shakespeare.lit"
            id="groupid"
            name="Group Name"
            type="system"
            members="100" />
    </query>
</iq>
```
* 如果不存在就返回错误
```xml
<iq from="groupchat.shakespeare.lit"
  id="lx09df27"
  to="hag66@shakespeare.lit/pda"
  type="error">
  <query xmlns="http://jabber.org/protocol/groupchat#groupinfo">
    <error by="groupchat.shakespeare.lit" type="cancel">
      <not-exists xmlns="urn:ietf:params:xml:ns:xmpp-stanzas"/>
    </error>
  </query>
</iq>
```

##查询群组成员
* 查询指定群组的成员列表
```xml
<iq from="crone1@shakespeare.lit/desktop"
    id="member3"
    to="groupid@groupchat.shakespeare.lit"
    type="get">
    <query xmlns="http://jabber.org/protocol/groupchat#members" />
</iq>
```
* 返回指定群组成员列表
```xml
<iq from="coven@chat.shakespeare.lit"
    id="member3"
    to="crone1@shakespeare.lit/desktop"
    type="result">
    <query xmlns="http://jabber.org/protocol/groupchat#members">
        <item jid="hag66@shakespeare.lit"
              mid="someone-id-1"
              nick="thirdwitch"
              role="owner"/>
        <item jid="hag66@shakespeare.lit"
              mid="someone-id-2"
              nick="thirdwitch1"
              role="member"/>
    </query>
</iq>
```

#拥有者用例
##创建群组
* 查询群组是否存在
```xml
<iq from="hag66@shakespeare.lit/pda"
  id="lx09df27"
  to="groupchat.shakespeare.lit"
  type="get">
  <query xmlns="http://jabber.org/protocol/groupchat#groupinfo">
    <property name="name" value="The Group Name" />
  </query>
</iq>
```
* 如果存在返回指定名字的群信息
```xml
<iq from="groupchat.shakespeare.lit"
  id="lx09df27"
  to="hag66@shakespeare.lit/pda"
  type="result">
  <query xmlns="http://jabber.org/protocol/groupchat#groupinfo">
    <item jid="groupid@groupchat.shakespeare.lit"
      id="groupid"
      name="Group Name"
      type="customized"
      members="100" />
  </query>
</iq>
```
* 如果不存在返回错误码
```xml
<iq from="groupchat.shakespeare.lit"
  id="lx09df27"
  to="hag66@shakespeare.lit/pda"
  type="result">
  <query xmlns="http://jabber.org/protocol/groupchat#groupinfo">
    <error by="groupchat.shakespeare.lit" type="cancel">
      <not-exists xmlns="urn:ietf:params:xml:ns:xmpp-stanzas"/>
    </error>
  </query>
</iq>
```
* 如果群组不存在,则创建改群,并且可以在创建的同时指定成员
```xml
<presence
    from="crone1@shakespeare.lit/desktop"
    to="groupid@groupchat.shakespeare.lit">
    <x xmlns="http://jabber.org/protocol/groupchat#creategroup"/>
    <properties>
      <property name="name">the group name</property>
      <property name="members">
        <item jid="someone@shakespeare.lit/pda"
                nick="nick name"
                mid="memberid"
                role="member" />
      </property>
    </properties>
</presence>
```
* 广播群组信息给每一个群组成员
```xml
<presence
  from="groupid@groupchat.shakespeare.lit"
  to="crone1@shakespeare.lit/desktop">
  <x xmlns="http://jabber.org/protocol/groupchat#creategroup"/>
  <properties>
    <property name="name">the group name</property>
    <property name="id">groupid</property>
  </properties>
</presence>
```

* 如果创建失败返回错误码
```xml
<presence
    from="groupid@groupchat.shakespeare.lit/thirdwitch"
    to="hag66@shakespeare.lit/pda"
    type="error">
    <x xmlns="http://jabber.org/protocol/groupchat#creategroup"/>
    <error by="groupid@groupchat.shakespeare.lit" type="cancel">
        <not-allowed xmlns="urn:ietf:params:xml:ns:xmpp-stanzas"/>
    </error>
</presence>
```

##修改群组信息
* 请求修改群组信息
```xml
<presence
  from="crone1@shakespeare.lit/desktop"
  to="groupid@groupchat.shakespeare.lit">
  <x xmlns="http://jabber.org/protocol/groupchat#updategroup"/>
  <properties>
    <property name="name">the group name</property>
  </properties>
</presence>
```
* 如果修改成功返回群组信息
```xml
<presence
  from="groupid@groupchat.shakespeare.lit"
  to="crone1@shakespeare.lit/desktop">
  <x xmlns="http://jabber.org/protocol/groupchat#updategroup"/>
  <properties>
    <property name="name">the group name</property>
    <property name="id">groupid</property>
  </properties>
</presence>
```
并且群组内广播改变
```xml
<presence
  from="groupid@groupchat.shakespeare.lit"
  to="crone1@shakespeare.lit/desktop">
  <x xmlns="http://jabber.org/protocol/groupchat#updategroup"/>
  <properties>
    <property name="name">the group name</property>
  </properties>
</presence>
```
* 如果修改失败返回错误码
```xml
<presence
  from="groupid@groupchat.shakespeare.lit/thirdwitch"
  to="hag66@shakespeare.lit/pda"
  type="error">
  <x xmlns="http://jabber.org/protocol/groupchat#updategroup"/>
  <error by="groupid@groupchat.shakespeare.lit" type="cancel">
    <not-allowed xmlns="urn:ietf:params:xml:ns:xmpp-stanzas"/>
  </error>
</presence>
```
##删除群组
* 请求删除指定群组
```xml
<presence
  from="crone1@shakespeare.lit/desktop"
  to="groupid@groupchat.shakespeare.lit">
  <x xmlns="http://jabber.org/protocol/groupchat#deletegroup"/>
</presence>
```
* 通知所有群成员注销指定的群
```xml
<presence
  from="groupid@groupchat.shakespeare.lit"
  to="crone1@shakespeare.lit/desktop">
  <x xmlns="http://jabber.org/protocol/groupchat#deletegroup"/>
</presence>
```
* 删除失败就直接返回错误
```xml
<presence
  from="groupid@groupchat.shakespeare.lit/thirdwitch"
  to="hag66@shakespeare.lit/pda"
  type="error">
  <x xmlns="http://jabber.org/protocol/groupchat#groupdelete"/>
  <error by="groupid@groupchat.shakespeare.lit" type="cancel">
    <not-allowed xmlns="urn:ietf:params:xml:ns:xmpp-stanzas"/>
  </error>
</presence>
```
##添加成员
* 请求添加指定成员到指定群组
```xml
<presence
  from="crone1@shakespeare.lit/desktop"
  to="groupid@groupchat.shakespeare.lit">
  <x xmlns="http://jabber.org/protocol/groupchat#addmember"/>

  <item jid="crone1@shakespeare.lit/desktop"
    nick="nick name"
    mid="memberid"
    role="owner" />
</presence>
```
* 通知所有群成员添加新成员
```xml
<presence
  from="groupid@groupchat.shakespeare.lit"
  to="crone1@shakespeare.lit/desktop">
  <x xmlns="http://jabber.org/protocol/groupchat#addmember"/>

  <item jid="crone1@shakespeare.lit/desktop"
    nick="nick name"
    mid="memberid"
    role="owner" />
</presence>
```
* 添加失败就直接返回错误
```xml
<presence
  from="groupid@groupchat.shakespeare.lit/thirdwitch"
  to="hag66@shakespeare.lit/pda"
  type="error">
  <x xmlns="http://jabber.org/protocol/groupchat#groupdelete"/>
  <error by="groupid@groupchat.shakespeare.lit" type="cancel">
    <not-allowed xmlns="urn:ietf:params:xml:ns:xmpp-stanzas"/>
  </error>
</presence>
```
##删除成员
* 请求删除指定成员到指定群组
```xml
<presence
  from="crone1@shakespeare.lit/desktop"
  to="groupid@groupchat.shakespeare.lit">
  <x xmlns="http://jabber.org/protocol/groupchat#addmember"/>

  <item jid="crone1@shakespeare.lit/desktop"
    nick="nick name"
    mid="memberid"
    role="owner" />
</presence>
```
* 通知所有群成员删除成员
```xml
<presence
  from="groupid@groupchat.shakespeare.lit"
  to="crone1@shakespeare.lit/desktop">
  <x xmlns="http://jabber.org/protocol/groupchat#addmember"/>

  <item jid="crone1@shakespeare.lit/desktop"
    nick="nick name"
    mid="memberid"
    role="owner" />
</presence>
```
* 删除失败就直接返回错误
```xml
<presence
  from="groupid@groupchat.shakespeare.lit/thirdwitch"
  to="hag66@shakespeare.lit/pda"
  type="error">
  <x xmlns="http://jabber.org/protocol/groupchat#groupdelete"/>
  <error by="groupid@groupchat.shakespeare.lit" type="cancel">
    <not-allowed xmlns="urn:ietf:params:xml:ns:xmpp-stanzas"/>
  </error>
</presence>
```

##成员用例
##发送群组消息
* 用户发给群组组件
```xml
<message
  from="hag66@shakespeare.lit/pda"
  id="hysf1v37"
  to="groupid@groupchat.shakespeare.lit"
  type="groupchat">
  <body>Harpier cries: "tis time, "tis time.</body>
</message>
```
* 群组组件分发给所有成员
```xml
<message
  from="groupid@groupchat.shakespeare.lit/memberid"
  id="hysf1v37"
  to="hag66@shakespeare.lit/pda"
  type="groupchat">
  <body>Harpier cries: "tis time, "tis time.</body>
</message>
```
* 成员收到消息,反馈给发送者
```xml
<message
    from="groupid@groupchat.shakespeare.lit/memberid"
    to="hag66@shakespeare.lit/pda"
    type="groupchat">
    <read xmlns="http://jabber.org/protocol/chatstates"/>
</message>
```

##@指定用户
* 用户发给群组组件
```xml
<message
  from="hag66@shakespeare.lit/pda"
  id="hysf1v37"
  to="groupid@groupchat.shakespeare.lit"
  type="groupchat">
  <body>Harpier cries: "tis time, "tis time.</body>
  <x xmlns="jabber:x:data" type="at" id="hag66@shakespeare.lit/pda" />
</message>
```
* 群组组件分发给所有成员
```xml
<message
  from="groupid@groupchat.shakespeare.lit/memberid"
  id="hysf1v37"
  to="hag66@shakespeare.lit/pda"
  type="groupchat">
  <body>Harpier cries: "tis time, "tis time.</body>
  <x xmlns="jabber:x:data" type="at" id="hag66@shakespeare.lit/pda" />
</message>
```
* 成员收到消息,反馈给发送者
```xml
<message
  from="groupid@groupchat.shakespeare.lit/memberid"
  to="hag66@shakespeare.lit/pda"
  type="groupchat">
  <read xmlns="http://jabber.org/protocol/chatstates"/>
</message>
```

##查看成员信息
```xml
<iq from="hag66@shakespeare.lit/pda"
  id="member3"
  to="groupid@groupchat.shakespeare.lit"
  type="get">
  <query xmlns="http://jabber.org/protocol/groupchat#memberinfo">
    <property name="nick" value="thirdwitch"/>
  </query>
</iq>
```
```xml
<iq from="groupid@groupchat.shakespeare.lit"
  id="member3"
  to="hag66@shakespeare.lit/pda"
  type="result">
  <query xmlns="http://jabber.org/protocol/groupchat#memberinfo">
    <item jid="hag66@shakespeare.lit"
      mid="someone-id-1"
      nick="thirdwitch"
      role="owner"/>
  </query>
</iq>
```
状态码

业务角色
实现注意事项
