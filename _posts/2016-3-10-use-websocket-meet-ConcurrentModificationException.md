---
layout : post
title : '使用websocket时遇到的concurrentModificationException'
category : java
tags : ['java','websocket']
---

一个使用websocket来推送数据的项目，在修改了css的情况下，刷新页面来看看效果的时候碰到了。。
然后就出现了这个错误。
主要思路是建立一个List来保存session，推送的时候遍历session推送数据。


具体代码如下:

```
	private static final List<Session> WEB_SOCKET_SESSIONS = new ArrayList<Session>();
	
    	@OnOpen
	public void onOpen(Session session, EndpointConfig config) {
		WEB_SOCKET_SESSIONS.add(session);
		logger.info("WebSocket 客户端 " + session.getId() + " 已连接, 当前连接数：" + WEB_SOCKET_SESSIONS.size());
	}

	@OnClose
	public void onClose(Session session) {
		WEB_SOCKET_SESSIONS.remove(session);
		logger.info("WebSocket 客户端 " + session.getId() + " 被关闭, 当前连接数：" + WEB_SOCKET_SESSIONS.size());
	}
    
  public void push(Object data){
 for (Session s : WEB_SOCKET_SESSIONS) {
				try {
					s.getBasicRemote().sendText(JSON.toJSONString(data));
				} catch (IOException e) {
					e.printStackTrace();
				}
			}	
            }
```


然后思考了一下，每次刷新页面都会导致List移除一次，再添加一次。自然也就改变了list的内容，所以导致了fail-fast.

修改之后的代码如下:


```
	private static void push(Object content,String type){
		Map<String,Object> data = new HashMap<String,Object>();
		data.put("type", type);
		data.put("data", content);
		try{
			for (Session s : WEB_SOCKET_SESSIONS) {
				try {
					s.getBasicRemote().sendText(JSON.toJSONString(data));
				} catch (IOException e) {
					e.printStackTrace();
				}
			}	
		}catch(ConcurrentModificationException e){
			//什么也不做，因为前台刷新或者关闭页面，导致sessions数量变化
		}
		
	}
```
