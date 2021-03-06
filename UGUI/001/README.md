# Unity UGUI 按钮绑定事件的 4 种方式

UGUI 可视化创建以及关联事件很方便, 动态创建可以利用创建好的 Prefab 进行实例化, 只是在关联事件上有些复杂, 本文总结了几种给按钮绑定事件的关联方式.

## 1. 可视化创建及事件绑定 # #

Step 1 : 通过 Hierarchy 面板创建 `UI > Button`.

![](http://wx4.sinaimg.cn/mw690/a53846c3gy1fcfo8xobhwj20dw07cglx.jpg)

Step 2 : 创建一个脚本 TestClick.cs, 定义了一个 Click 的 public 方法.

Step 3 : 选中 Hierarchy 中的 Button, `Add Component` 脚本 TestClick.cs

Step 4 : 在 `Button(Script)` 关联 TestClick 脚本里的 Click 方法.

![](http://wx1.sinaimg.cn/mw690/a53846c3gy1fcfo8wu4uyj20lm0lw76q.jpg)

Step 5 : Done.

TestClick.cs

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class TestClick : MonoBehaviour {

	public void Click(){
		Debug.Log ("Button Clicked. TestClick.");
	}
}

```

## 2. 通过直接绑定脚本来绑定事件 # #

Step 1 : 通过 Hierarchy 面板创建 `UI > Button`.

Step 2 : 创建一个 ClickHandler.cs 脚本, 定义了一个私有方法 OnClick(), 并在 Start() 方法里为 Button 添加点击事件的监听,作为参数传入 OnClick 方法.

Step 3 : 将 ClickHandler 绑定在 Button 对象上.

![](http://wx3.sinaimg.cn/mw690/a53846c3gy1fcfo8wxoijj20lo09kq3l.jpg)

Step 4 : Done.

ClickHandler.cs

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class ClickHandler : MonoBehaviour {

	void Start () {
		Button btn = this.GetComponent<Button> ();
		btn.onClick.AddListener (OnClick);
	}

	private void OnClick(){
		Debug.Log ("Button Clicked. ClickHandler.");
	}

}

```

## 3. 通过 EventTrigger 实现按钮点击事件 # #

UGUI 系统中 Button 默认只提供了 OnClick 的调用方法, 有时候我们还需要监听鼠标进入事件 (MouseIn) 和鼠标滑出事件 (MouseOut). 就需要借助 UI 系统中的 EventTrigger 脚本来实现.

Step 1 : 通过 Hierarchy 面板创建 `UI > Button`.

Step 2 : 创建一个 EventTriggerHandler.cs 脚本, 利用 UnityEngine.EventSystems.EventTrigger 添加监听事件.

Step 3 : 绑定 EventTriggerHandler.cs 脚本到 Button 上.

![](http://wx1.sinaimg.cn/mw690/a53846c3gy1fcfo8wunp4j20lo0c4wfj.jpg)

Step 4 : Done.


EventTriggerHandler.cs

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.EventSystems;

// 需要 EventTrigger 脚本的支援
[RequireComponent(typeof(UnityEngine.EventSystems.EventTrigger))]
public class EventTriggerHandler : MonoBehaviour {

	// Use this for initialization
	void Start () {
		Button btn = this.GetComponent<Button> ();
		EventTrigger trigger = btn.gameObject.GetComponent<EventTrigger> ();
		EventTrigger.Entry entry = new EventTrigger.Entry ();
		// 鼠标点击事件
		entry.eventID = EventTriggerType.PointerClick;
		// 鼠标进入事件 entry.eventID = EventTriggerType.PointerEnter;
		// 鼠标滑出事件 entry.eventID = EventTriggerType.PointerExit;
		entry.callback = new EventTrigger.TriggerEvent ();
		entry.callback.AddListener (OnClick);
		// entry.callback.AddListener (OnMouseEnter);
		trigger.triggers.Add (entry);
	}

	private void OnClick(BaseEventData pointData){
		Debug.Log ("Button Clicked. EventTrigger..");
	}

	private void OnMouseEnter(BaseEventData pointData){
		Debug.Log ("Button Enter. EventTrigger..");
	}
}

```

## 4. 通过 MonoBehaviour 实现事件类接口来实现事件的监听 # #

Step 1 : 通过 Hierarchy 面板创建 `UI > Button`.

Step 2 : 创建一个 EventHandler.cs 脚本.

Step 3 : 将脚本绑定在 Button 对象上.

![](http://wx2.sinaimg.cn/mw690/a53846c3gy1fcfo8wb4dej20lo09a74w.jpg)

Step 4 : Done.

EventHandler.cs

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.EventSystems;

public class EventHandler : MonoBehaviour, IPointerClickHandler, IPointerEnterHandler, IPointerExitHandler, IPointerDownHandler, IDragHandler {

	public void OnPointerClick(PointerEventData eventData){
		if(eventData.pointerId == -1){
			Debug.Log ("Left Mouse Clicked.");
		} else if(eventData.pointerId == -2){
			Debug.Log ("Right Mouse Clicked.");
		}
	}

	public void OnPointerEnter(PointerEventData eventData){
		Debug.Log ("Pointer Enter..");
	}

	public void OnPointerExit(PointerEventData eventData){
		Debug.Log ("Pointer Exit..");
	}

	public void OnPointerDown(PointerEventData eventData){
		Debug.Log ("Pointer Down..");
	}

	public void OnDrag(PointerEventData eventData){
		Debug.Log ("Dragged..");
	}

}

```

UGUI  如何判断 UI 元素被点击时是鼠标的哪个按键, 上面的代码中我们可以根据 eventData.pointerId 来监听是鼠标左键还是右键. 但是每个 UI 元素都创建一个 MonoBehaviour 来监听各个事件显然不好, 下面是通过利用 Delegate 和 Event 来做一个通用类 UIEventListener 来处理事件 (观察者模式).

UIEventListener.cs

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.EventSystems;

public class UIEventListener : MonoBehaviour, IPointerClickHandler, IPointerEnterHandler, IPointerExitHandler {

	// 定义事件代理
	public delegate void UIEventProxy(GameObject gb);

	// 鼠标点击事件
	public event UIEventProxy OnClick;

	// 鼠标进入事件
	public event UIEventProxy OnMouseEnter;

	// 鼠标滑出事件
	public event UIEventProxy OnMouseExit;

	public void OnPointerClick(PointerEventData eventData){
		if (OnClick != null)
			OnClick (this.gameObject);
	}

	public void OnPointerEnter(PointerEventData eventData){
		if (OnMouseEnter != null)
			OnMouseEnter (this.gameObject);
	}

	public void OnPointerExit(PointerEventData eventData){
		if (OnMouseExit != null)
			OnMouseExit (this.gameObject);
	}

}

```

TestEvent.cs

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class TestEvent : MonoBehaviour {

	void Start () {
		Button btn = this.GetComponent<Button> ();
		UIEventListener btnListener = btn.gameObject.AddComponent<UIEventListener> ();

		btnListener.OnClick += delegate(GameObject gb) {
			Debug.Log(gb.name + " OnClick");
		};

		btnListener.OnMouseEnter += delegate(GameObject gb) {
			Debug.Log(gb.name + " OnMouseEnter");
		};

		btnListener.OnMouseExit += delegate(GameObject gb) {
			Debug.Log(gb.name + " OnMOuseExit");
		};
	}

}

```

TestEvent 脚本绑定在 Button 上即可.
![](http://wx2.sinaimg.cn/mw690/a53846c3gy1fcfo8whtpoj20lq09eq3k.jpg)

---

Project 结构

![](http://wx2.sinaimg.cn/mw690/a53846c3gy1fcfo8w7btyj20dw09k755.jpg)

代码 : ↑↑↑ UGUI4Event.unitypackage

End.
