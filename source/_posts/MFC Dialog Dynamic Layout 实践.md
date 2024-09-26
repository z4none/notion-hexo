---
updated: '2018-10-18 08:00:00'
categories: code
excerpt: "今天接触到一个 MFC Dialog 项目，对于一个 resizeable 的对话框，控件的布局一直是个棘手的问题，由于 MFC 框架较老并且为了保持向前兼容，所以一直没有提供这方面的支持，直到 VisualStudio 2015 在 MFC 中引进了 Dynamic Layout，关于 Dynamic Layout 的说明可以参见 MSDN 的 blog :\_MFC Dynamic Dialog Layout\n。"
date: '2018-10-18 08:00:00'
tags:
  - C++
urlname: mfc-dynamic-layout
title: MFC Dialog Dynamic Layout 实践
---

今天接触到一个 MFC Dialog 项目，对于一个 resizeable 的对话框，控件的布局一直是个棘手的问题，由于 MFC 框架较老并且为了保持向前兼容，所以一直没有提供这方面的支持，直到 VisualStudio 2015 在 MFC 中引进了 Dynamic Layout，关于 Dynamic Layout 的说明可以参见 MSDN 的 blog : [MFC Dynamic Dialog Layout](https://blogs.msdn.microsoft.com/vcblog/2015/04/29/mfc-dynamic-dialog-layout/)。


简单地说就是可以在 VisualStudio 的资源编辑器中设置控件的 Dynamic Layout 的 Moving 和 Sizing 属性实现控件的对其和按比例缩放。


但是如果只设置控件的 Dynamic Layout 属性会出现如下问题，当窗口缩小到小于初始大小时，控件会被挤出显示范围：


![20190830163520.gif](https://s.z4none.me/blog/16ed5dcb557ec702ae00e2a242a6ca66.gif)


通过跟踪 Dynamic Layout 功能的源代码到 wincore2.cpp 中可以看到 `CMFCDynamicLayout::GetHostWndRect` 的定义有：


```c++
rect.right = rect.left + max(m_MinSize.cx, rect.Width());
rect.bottom = rect.top + max(m_MinSize.cy, rect.Height());

```


控件的大小受到 m_MinSize 的大小的制约，这个 m_MinSize 即为对话框初始化时的大小。


现在知道了原因，那么有两个解决办法：


## 1. 通过 OnGetMinMaxInfo 限制对话框的大小不能小于初始大小


这样需要在对话框设计时将其缩放到最小尺寸，然后通过定义 WM_GETMINMAXINFO 消息函数确保其不会进一步缩小：


```c++
//
void CMyDlg::OnGetMinMaxInfo(MINMAXINFO * lpMMI)
{
	CMFCDynamicLayout * layout = GetDynamicLayout();

	if (layout)
	{
		CSize size = layout->GetMinSize();
		CRect rect(0, 0, size.cx, size.cy);
		AdjustWindowRect(&rect, GetStyle(), FALSE);
		lpMMI->ptMinTrackSize.x = rect.Width();
		lpMMI->ptMinTrackSize.y = rect.Height();
	}

	CDialogEx::OnGetMinMaxInfo(lpMMI);
}

```


其中 CMFCDynamicLayout::GetMinSize() 获取到的即为对话框的初始大小。


## 2. 通过 SetMinSize 设置初始大小为 CSize(0, 0)


设置对话框初始大小为 CSize(0, 0)， 只要对话框大小大于 CSize(0, 0) 就不会出现控件被挤出窗口的问题


```c++
BOOL CMylDlg::OnInitDialog()
{
    ...

    CMFCDynamicLayout * layout = GetDynamicLayout();
	layout->SetMinSize(CSize(0, 0));
}

```


当然以上两个方法可以结合起来， 在 OnInitDialog 中设置窗口最小大小为一个指定值，而不使用资源编辑器中设计的大小。


![20190830163956.gif](https://s.z4none.me/blog/23d02ae93f4f1c80a2b7820804331245.gif)


## WTL


值得一提的是，在 2018 年 WTL 也对 DynamicLayout 也提供了支持，将对话框继承于 CDynamicDialogLayout ，并在 OnInitDialog 中调用 InitDynamicLayout 即可


```c++
class CMainDlg :
	public CDynamicDialogLayout<CMainDlg>
	...
{
	// void InitDynamicLayout(bool bAddGripper = true, bool bMinTrackSize = true)
	LRESULT OnInitDialog(UINT /*uMsg*/, WPARAM /*wParam*/, LPARAM /*lParam*/, BOOL& /*bHandled*/)
	{
		//
		//
		InitDynamicLayout(true, true);
	}
}


```


其中 InitDynamicLayout 的参数含义为：

- bAddGripper 是否显示 Gripper，右下的拖动手柄
- bMinTrackSize 是否限定窗口最小尺寸
