使用Node-FFI 调用 Win32 API 支持Elctron
maxcalibur
maxcalibur
长期求职⁄(⁄ ⁄ ⁄ω⁄ ⁄ ⁄)⁄非米忽悠勿扰
5 人赞同了该文章
1. 前言
修改中。。。

其实今天的主角并不是Electron, 而是Javascript在windows上的一些拓展，那么下面就结合我使用ffi封装的库来简单介绍下 如何使用js调用win32 Api ,

GitHub - deskbtm/win-win-api: win32 api binding for js/ts that powered by node-ffi
​github.com/deskbtm/win-win-api

2. 使用
win-win-api 包含了 User32，Kernal32，Comctl32（部分）对于其他的lib请自行添加

WinWin({unicode?:boolean})
import {WinWin} from 'win-win-api';
const winwin = new WinWin();

// winwin.user32();
// winwin.kernel32();

const winFns = winwin.winFns(); // 包含了 user32 and kernel32 comctl32
win32 中可以使用unicode编码， 也可以使用ascii, 一般像 以W结尾的函数就是以UNICODE 编码，A结尾则是ASCII ， Ex 结尾是对 原有函数的拓展


在win-win 中去掉了FindWindow 这种函数

L, _T, _TEXT_ 和c++ 中一样 把字符转换成宽字节
使用Buffer.from(xxx, 'ucs2') 实现 使字符串中的每个字符都会使用 2 个或 4 个字节进行编码

const clsName = "demo";
winFns.FindWindowW(clsName, 0);  //报错

winFns.FindWindowW(L(clsName), 0);
CPP 中同样

PLCSTR clsName = "demo";

FindWindowW(clsName, 0);  //报错
PLCSTR name = L"demo";

FindWindowW(name, 0);
Ref win-win-api 覆盖了 ts 类型 原有的@types/ref-napi 丢失接口
"ref" documentation v0.3.3
​tootallnate.github.io/ref/
常用： refType 类型的指针 一般用于结构体 共同体，ref 指针, deref 解引用, address 获取地址 下面有具体例子

ffi
https://github.com/node-ffi/node-ffi/wiki/Node-FFI-Tutorial
​github.com/node-ffi/node-ffi/wiki/Node-FFI-Tutorial
Struct
import { ref, StructType, CPP, TS } from 'win-win-api';

const Struct = StructType(ref);

const MSG:TS.MSG  = Struct({
 hwnd: CPP.HWND,
 message: CPP.UINT,
 wParam: CPP.WPARAM,
 lParam: CPP.LPARAM,
 time: CPP.DWORD,
 pt: CPP.POINT
});

const msg = new MSG();
console.log(msg.ref()); 获取一个buffer类型 具体参考ref

CPP 包含了大部分win cpp的类型如 CPP.HWND 和 一些常用的常量 CPP.WM_MOUSEFIRST = 0x0200;
以CPP.StructXXXX开头的是分装好的结构体

这里解释下windows最常见的概念 句柄 ，其实可以看成一个结构体，包含了一些信息

就像

var hwnd = {
id: 12232,
width: '100px',
height: '100px'
}
TS Ts类型 与CPP类型一致


自定义自己的方法

RefStruct 用于结构体类型

注意
win-win 不可能包含整个win32 api 所以你就需要自己实现了
import {ffi} from "win-win-api";

interface XXXFns{
oooFns: (demo:TS.DWROD)=>TS.HWND
} 

xxxFns: TsWin32Fns<XXXFns> = ffi.Library('库', {
函数名: [返回类型,[...参数类型]]
});

xxxFns.oooFns(1234);
2. C ++的某些参数类型是不确定的，这时候你需要创建一个新文件并重写一个适合你的函数

overwrite.ts

import { CPP, ref } from 'win-win';
export const customFns = {
 CallNextHookEx: [CPP.LRESULT, [CPP.HHOOK, CPP.INT, CPP.WPARAM, ref.refType(CPP.MOUSEHOOKSTRUCT)]]
};
// 必须要执行 可以在其他文件 import "overwrite";
WinWin.overwrite({ user32Fns: customFns })
3. ffi 没有能力操作 宏定义的函数，通常会报 Symbol winXXX 错误，所以win-win 取消了所有#define 的方法 但像 MAKELPARMA 是可以使用的 具体看 cpp\user\user_macro.ts

ffi.Library('User32', {
 GetMessage: [...] //这是肯定会报错的
});
但也有解决的办法，就是对着源文件自己实现

CPP



TS


4. win-win 之实现了 comctl小部分， 里面绝大部分是#define 但是你可以使用SendMessageW 去实现

5. win-win 没有包含数组，和共同体 可以使用自行完成

ref-array-di
​www.npmjs.com/package/ref-array-di

ref-union-di
​www.npmjs.com/package/ref-union-di

6. electron 中

const bufferCastInt32 = function (buf: Buffer): number {
	return os.endianness() == "LE" ?
		buf.readInt32LE() : buf.readInt32BE();
};

electron 中

const hwnd:number = bufferCastInt32(mainWindow.getNativeHandle() // 获得electron 窗口句柄);

SetParent(hwnd, GetWindowDesktop());
e.g.
创建全局鼠标钩子
import { CPP, ref, WinWin, ffi } from 'win-win';
export const customFns = {
 CallNextHookEx: [CPP.LRESULT, [CPP.HHOOK, CPP.INT, CPP.WPARAM, ref.refType(CPP.MOUSEHOOKSTRUCT)]]
};
// 必须要执行 可以在其他文件 import "overwrite";
WinWin.overwrite({ user32Fns: customFns })

const {
 CallNextHookEx,
 WindowFromPoint,
 SetWindowsHookExW,
 GetMessageW,
 DispatchMessageW,
 TranslateMessage,
 UnhookWindowsHookEx
} = new WinWin().user32();

const _createMouseHookProc = () => ffi.Callback(CPP.LRESULT,  // 相当于C++ 的CALLBACK
 [CPP.INT, CPP.WPARAM, ref.refType(CPP.StructMOUSEHOOKSTRUCT)],
 (nCode: TS.INT, wParam: TS.WPARAM, lParam: TS.RefStruct) => {

  const mouse: TS.MOUSEHOOKSTRUCT = lParam.deref();
  const pt = mouse.pt;
  const { x, y } = pt;
  const currentHwnd = WindowFromPoint(mouse.pt);

  return CallNextHookEx(0, nCode, wParam, lParam); // need overwrite
 }
)

const _mouseHook = SetWindowsHookExW(CPP.WH_MOUSE_LL, this._createMouseHookProc(), 0, 0);
const msg: TS.RefStruct = new CPP.StructMSG();

while (GetMessageW(msg.ref(), 0, 0, 0) && this._trigger) {
 TranslateMessage(msg.ref());
 DispatchMessageW(msg.ref());
}

UnhookWindowsHookEx(_mouseHook);
2. 创建线程 (一般使用_beginThreadEx)

const { WinWin, ffi, CPP, L, NULL } = require('win-win-api');

const { CreateThread, MessageBoxW } = new WinWin().winFns();

const proc = ffi.Callback(CPP.INT, [CPP.PVOID], () => {
	MessageBoxW(0, L("exmpale"), null, CPP.MB_OK | CPP.MB_ICONEXCLAMATION);
});

CreateThread(null, 0, proc, NULL, 0, NULL);
3. 在创建的新建线程中创建全局鼠标钩子 e.g.


注意: 在Electron 中通过CreateThread创建的鼠标HOOK会导致CPU占用暴增。

Win32 Api Microsoft Doc
https://docs.microsoft.com/en-us/windows/win32/
​docs.microsoft.com/en-us/windows/win32/