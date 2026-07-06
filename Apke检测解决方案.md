% 中文文档，直接复制到TeX编辑器即可使用
\documentclass{article}
\usepackage{ctex}
\begin{document}

\centering
\Large{\textbf{ApkeSU 过环境完整操作文档}}
\vspace{10pt}
\hrule
\vspace{5pt}

\section{一、说明}
检测代码已经提前屏蔽3类检测：挂载风险、5555调试端口、和平精英风控词条。
按照下面步骤操作，就能清除剩下所有风险；全部设置完成后重启手机，再重新扫描检测工具。

\section{二、前置要求}
\begin{enumerate}
    \item 手机装好ApkeSU并正常激活，拥有完整ROOT权限
    \item 卸载MT管理器、LSPosed、太极、Frida、Xposed等工具、注入软件
    \item 只使用ApkeSU做隐藏配置，关闭hma、隐藏应用列表这类第三方隐藏工具
    \item 每完成一大项设置建议重启一次手机，全部操作做完最后再重启一次扫描
\end{enumerate}

\section{三、分项过检操作步骤}

\subsection{1. 消除SELinux底层风险词条}
\begin{enumerate}
    \item 打开ApkeSU，底部切换【设置】→【Root功能】
    \item 找到「隐藏SELinux修改」，打开开关
    \item 保存设置，重启手机
\end{enumerate}
作用：拦截读取 /sys/fs/selinux/enforce 文件，不再弹出SELinux宽松、读取失败相关风险

\subsection{2. 消除SoterService证书风险词条}
\begin{enumerate}
    \item ApkeSU进入【应用黑名单/隐藏列表】
    \item 添加系统服务包名：com.qti.soter.service
    \item 勾选：隐藏应用本体、屏蔽目录 /system_ext/app/SoterService
    \item 设置自动写入文件 /data/adb/apkesu/hide_blacklist
\end{enumerate}
作用：检测读取黑名单识别服务已隐藏，不会再提示TEE环境不可信

\subsection{3. 消除ADB调试相关风险}
\begin{enumerate}
    \item 设置→关于本机，连续点击版本号打开开发者选项
    \item 关闭「USB调试」「无线ADB调试」，删除全部无线配对设备
    \item 打开终端，输入ROOT命令清空端口：
\begin{verbatim}
su
setprop service.adb.tcp.port -1
setprop persist.adb.tcp.port -1
stop adbd && start adbd
\end{verbatim}
    \item ApkeSU【Root功能】打开「屏蔽ADB调试痕迹」
\end{enumerate}

\subsection{4. 清除ROOT框架目录风险}
\begin{enumerate}
    \item ApkeSU【模块隐藏】开启全局目录伪装
    \item 需要隐藏目录：/magisk、/dev/magisk、/debug_ramdisk、/data/adb/
    \item 开启服务伪装，屏蔽 magiskd、init.svc.magisk 系统属性读取
    \item 开启su隐藏，阻止命令检索到su二进制文件
\end{enumerate}

\subsection{5. 消除LSPosed/Riru/Zygisk注入风险}
\begin{enumerate}
    \item 彻底卸载LSPosed、EdXposed、Riru、Zygisk全部模块
    \item ApkeSU屏蔽路径：
    /data/adb/lspd
    /data/misc/riru
    /data/adb/modules/zygisk_lsposed
    \item 屏蔽属性 ro.riru.version、ro.xposed.version
\end{enumerate}

\subsection{6. 解决机型品牌/型号篡改风险}
\begin{enumerate}
    \item ApkeSU打开【机型伪装】
    \item 填写手机原厂真实品牌、型号，和系统原始参数一致
    \item 锁定 ro.product.brand、ro.product.model，屏蔽篡改痕迹读取
\end{enumerate}

\subsection{7. 第三方自定义ROM系统风险处理}
\begin{enumerate}
    \item ApkeSU【系统属性伪装】
    \item 修改参数为官方原厂内容：
    ro.build.flavor
    ro.build.display.id
    ro.build.fingerprint
    \item 清除test-keys测试签名标识，替换为官方release签名
\end{enumerate}

\subsection{8. 消除GKI通用内核风险}
ApkeSU打开【内核伪装】，拦截ro.boot.kernel、系统版本里gki关键词读取

\subsection{9. 模拟器/云手机虚拟环境风险处理}
\begin{enumerate}
    \item 关闭所有虚拟机、云手机类APP
    \item ApkeSU屏蔽特征：
    ro.kernel.qemu
    硬件标识 goldfish / ranchu / vbox
\end{enumerate}

\subsection{10. 零散遗留风险清理}
\begin{enumerate}
    \item Keybox痕迹：ApkeSU屏蔽 /dev/keybox 相关文件
    \item ACE日志清理，终端执行：rm -f /data/local/tmp/ace_scan_trace.log
    \item 删除DeltaACE模块文件夹 /data/adb/delta_ace_module
    \item 删除高危脚本 /data/local/tmp/crypt.sh、frida-server
\end{enumerate}

\section{四、最终验证流程}
\begin{enumerate}
    \item 全部设置完成，重启手机
    \item 关掉后台所有工具，清空检测APP缓存
    \item 打开检测工具重新扫描
    \item 正常结果：无任何高危风险词条
\end{enumerate}

\section{五、常见失败排查}
\begin{enumerate}
    \item SoterService依旧报错：打开 /data/adb/apkesu/hide_blacklist 确认包名已写入，保存后重启
    \item SELinux风险不消失：开关一次「隐藏SELinux修改」，等待配置写入后重启
    \item ADB调试残留：重复上面端口清空命令，删除全部开发者配对设备
    \item ROOT目录持续被检测：检查全部目录隐藏开关是否开启，无残留Magisk模块
    \item 多条风险残留：确认卸载全部Xposed、LSPosed、MT管理器类工具
\end{enumerate}

\end{document}
