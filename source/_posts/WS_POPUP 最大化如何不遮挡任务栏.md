---
updated: '2022-08-05 00:00:00'
categories: code
excerpt: 在桌面开发时，我们有时会创建样式为 WS_POPUP 的窗口，然后自绘标题栏。当窗口最大化时，会覆盖整个屏幕，盖住了任务栏，这通常不是我们想要的效果。
date: '2022-08-05 00:00:00'
tags:
  - C++
  - Windows
urlname: ws-popup-maxmize
title: WS_POPUP 最大化如何不遮挡任务栏
---

在桌面开发时，我们有时会创建样式为 `WS_POPUP` 的窗口，然后自绘标题栏。当窗口最大化时，会覆盖整个屏幕，盖住了任务栏，这通常不是我们想要的效果。


![1.gif](https://s.z4none.me/blog/512c58872c71c0e355ea3ca7126119da.gif)


为了避免遮挡任务栏，一般的做法是响应 `WM_GETMINMAXINFO` 消息，限制其最大化后的位置，代码如下：


```c++
void CwspopupmaximizeDlg::OnGetMinMaxInfo(MINMAXINFO* lpMMI)
{
    HMONITOR monitor = MonitorFromWindow(m_hWnd, MONITOR_DEFAULTTONULL);
    if (monitor)
    {
        // 获取窗口所在屏幕
        MONITORINFO info = { 0 };
        info.cbSize = sizeof(MONITORINFO);
        if (GetMonitorInfo(monitor, &info))
        {
            // 限制为工作区范围
            CRect rcWork = info.rcWork;
            lpMMI->ptMaxPosition = { 0, 0 };
            lpMMI->ptMaxSize.x = rcWork.Width();
            lpMMI->ptMaxSize.y = rcWork.Height();
        }
    }

    CDialogEx::OnGetMinMaxInfo(lpMMI);
}
```


写个测试程序、编译、运行，效果完美


![2.gif](https://s.z4none.me/blog/73acc171ca2975b9198d468f794e9e81.gif)


然而当多屏幕时，把测试程序放到副屏幕上，最大化后窗口不但遮住了任务栏，而且横向也超出屏幕，被截掉一部分。


![3.gif](https://s.z4none.me/blog/e57337ba209e521654e981d565f64b30.gif)


查阅 MSDN 中关于 **`MINMAXINFO`** **结构的描述**


> For systems with multiple monitors, the **ptMaxSize** and **ptMaxPosition** members describe the maximized size and position of the window on the primary monitor, even if the window ultimately maximizes onto a secondary monitor. In that case, the window manager adjusts these values to compensate for differences between the primary monitor and the monitor that displays the window. Thus, if the user leaves **ptMaxSize**  untouched, a window on a monitor larger than the primary monitor maximizes to the size of the larger monitor.


如果窗口位于第二屏，MINMAXINFO 中的坐标会进行一些 `转换` 然后再应用到第二屏上。但究竟是什么转换呢，MSDN 上没有明确指出。


经过搜寻，在微软 The Old New Thing 中找到了进一步的描述：


> **ptMax­Size** is trickier. If the specified size is greater than or equal to the size of the primary monitor, then the **ptMax­Size** is adjusted to include the difference in size between the primary monitor and the actual monitor. In other words, if **ptMax­Size** is 20 pixels larger than the primary monitor, then it will be adjusted to being 20 pixels larger than the actual monitor. But if **ptMax­Size** does not completely cover the monitor, then its value is used as-is.


> **ptMax­Size** 更为变态，如果它的值大于或者等于主屏幕，那么 **ptMax­Size** 调整时将考虑目标屏幕和主屏幕的差值。假如 **ptMax­Size**  的值比主屏幕大 20px， 那么它将被调整到比目标屏幕大 20px 的大小；如果 **ptMax­Size** 的值小于主屏幕， 那么它的值不变。


如此一来就能解释之前的问题了，我的主屏是 1920x1080, 第二屏是 2560x1440，当在第二屏上最大化时，得到的第二屏桌面大小为 2560x1410，超出了主屏大小，所以得按照差值方式调整。


要解决此问题，我们可以在 WM_SIZE 中判断 nType == SIZE_MAXIMIZED 时，修正窗口大小：


```c++
void CwspopupmaximizeDlg::OnSize(UINT nType, int cx, int cy)
{
    CDialogEx::OnSize(nType, cx, cy);

    if (nType == SIZE_MAXIMIZED)
    {
        SetDlgItemText(ID_MAXMIZE_RESTORE, L"restore");

        HMONITOR monitor = MonitorFromWindow(m_hWnd, MONITOR_DEFAULTTONULL);
        if (monitor)
        {
            MONITORINFO info = { 0 };
            info.cbSize = sizeof(MONITORINFO);
            if (GetMonitorInfo(monitor, &info))
            {
                CRect rcWork = info.rcWork;
                SetWindowPos(NULL, rcWork.left, rcWork.top, rcWork.Width(), rcWork.Height(), SWP_SHOWWINDOW);
            }
        }
    }
    else if (nType == SIZE_RESTORED)
    {
        SetDlgItemText(ID_MAXMIZE_RESTORE, L"maximize");
    }
    Invalidate();
} 
```



参考资料：


[**How does the window manager adjust ptMaxSize and ptMaxPosition for multiple monitors?**](https://devblogs.microsoft.com/oldnewthing/20150501-00/?p=44964)

