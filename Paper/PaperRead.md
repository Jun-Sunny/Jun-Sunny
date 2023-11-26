# 无感控制
## 基于模型
1. 拓展状态二阶非奇异终端转子磁链滑模观测器, 电机与控制学报. --在考虑参数摄动的情况下构建了超螺旋滑模观测器,有点晦涩.
2. 
## 基于高频脉冲注入
1. [硕士论文]基于高频方波注入永磁同步电机无位置传感器控制技术研究-李文涛-中国矿业大学
* 2.2节 PMSM的凸极效应分析;
* 公式4-16里面有错误,对$I\alpha h$和$I\beta h$展开表述有误;
2. 基于无滤波器方波信号注入的永磁同步电机初始位置检测方法-张国强
## 超螺旋滑模观测器
[硕士论文]基于改进锁相环的永磁同步电机无位置传感器控制研究-张甲哲
1. 一般化公式
$\begin{cases}\dfrac{d}{dt}\hat{x}_1=-k_1\left|x_1-\hat{x}_1\right|^{1/2}\operatorname{sign}(x_1-\hat{x}_1)+\hat{x}_2+\rho_1\\\\\dfrac{d}{dt}\hat{x}_2=-k_2\operatorname{sign}(x_1-\hat{x}_1)+\rho_2\end{cases}$

这里$\rho i$代表扰动项
根据李雅普诺夫稳定性证明,可以得到k1, k2的满足条件
$k_1>\delta_1|x_1|^{1/2}\text{,}k_2>k_1\frac{5\delta_1k_1+4\delta_1^2}{2(k_1-2\delta_1)}$

这里$\delta 1$需要满足
$\delta_1\geq\frac{\rho_1}{\left|x_1\right|^{1/2}}$

2. 将PMSM模型代入到一般模型中,可以得到
$\begin{cases}\frac{d}{dt}\hat{i}_\alpha=-\frac{R}{L_\mathrm{d}}\hat{i}_\alpha-\hat{\omega}_\mathrm{e}\frac{L_\mathrm{d}-L_\mathrm{q}}{L_\mathrm{d}}\hat{i}_\mathrm{\beta}+\frac{u_\mathrm{a}}{L_\mathrm{d}}-\frac{k_1}{L_\mathrm{d}}\big|\overline{i}_\alpha\big|^{1/2}\operatorname{sign}(\overline{i}_\alpha)-\frac{1}{L_\mathrm{d}}\int k_2\operatorname{sign}(\overline{i}_\alpha)dt\\\frac{d}{dt}\hat{i}_\mathrm{\beta}=-\frac{R}{L_\mathrm{d}}\hat{i}_\beta+\hat{\omega}_\mathrm{e}\frac{L_\mathrm{d}-L_\mathrm{q}}{L_\mathrm{d}}\hat{i}_\mathrm{\alpha}+\frac{u_\mathrm{\beta}}{L_\mathrm{d}}-\frac{k_1}{L_\mathrm{d}}\big|\overline{i}_\mathrm{\beta}\big|^{1/2}\operatorname{sign}(\overline{i}_\mathrm{p})-\frac{1}{L_\mathrm{d}}\int k_2\operatorname{sign}(\overline{i}_\mathrm{p})dt\end{cases}$

根据一般化公式,可以得到扰动项的满足条件为
$\begin{cases}\rho_1\left(\hat{i}_\alpha\right)=-\frac{R}{L_\mathrm{d}}\hat{i}_\alpha-\frac{\left(L_\mathrm{d}-L_\mathrm{q}\right)}{L_\mathrm{d}}\hat{o}_\mathrm{e}\hat{i}_\mathrm{\beta}+\frac{1}{L_\mathrm{d}}u_\alpha\leq\delta_\mathrm{l}\left|\hat{i}_\alpha\right|^{1/2}\\\rho_1\left(\hat{i}_\mathrm{\beta}\right)=-\frac{R}{L_\mathrm{d}}\hat{i}_\mathrm{\beta}+\frac{\left(L_\mathrm{d}-L_\mathrm{q}\right)}{L_\mathrm{d}}\hat{\omega}_\mathrm{e}\hat{i}_\alpha+\frac{1}{L_\mathrm{d}}u_\mathrm{\beta}\leq\delta_\mathrm{l}\left|\hat{i}_\mathrm{\beta}\right|^{1/2}&\end{cases}$

同时根据PMSM的公式,可以摘出反电动势项为
$\begin{cases}\hat e_\alpha=k_1\left|\tilde i_\alpha\right|^{1/2}\operatorname{sign}(\tilde i_\alpha)+k_2\int\operatorname{sign}(\tilde i_\alpha)dt\\\hat e_\beta=k_1\left|\tilde i_\alpha\right|^{1/2}\operatorname{sign}(\tilde i_\beta)+k_2\int\operatorname{sign}(\tilde i_\beta)dt&\end{cases}$

将扰动项近似为
$\begin{cases}\left|\rho_1(\hat{i}_\alpha)\right|\approx\left|\frac{u_\alpha}{L_\mathrm{d}}\right|\approx\frac{\omega_\mathrm{e}\psi_\mathrm{f}}{L_\mathrm{d}}\leq\delta_\mathrm{l}\left|\hat{i}_\alpha\right|^{1/2}\\\left|\rho_1(\hat{i}_\mathrm{\beta})\right|\approx\left|\frac{u_\mathrm{\beta}}{L_\mathrm{d}}\right|\approx\frac{\omega_\mathrm{e}\psi_\mathrm{f}}{L_\mathrm{d}}\leq\delta_\mathrm{l}\left|\hat{i}_\mathrm{\beta}\right|^{1/2}&\end{cases}$

作者使用转速来校正滑模观测器增益,从而形成自适应效果
$k_1=\mu_1\omega_e^*,\quad k_2=\mu_2\omega_e^{*2}$

并且根据之前得到的稳定性条件可以得到范围
$\begin{cases}\mu_1>2\lambda\\u_2>\mu_1\frac{5\lambda\mu_1+4\lambda^2}{2\mu_1-4\lambda}&\end{cases}$

这里$\lambda$应该要满足
$\left|\rho_1(\hat{i_\beta})\right|\approx\left|\frac{u_\mathrm{\beta}}{L_\mathrm{d}}\right|\approx\frac{\omega_\mathrm{e}\psi_\mathrm{f}}{L_\mathrm{d}}\leq\lambda\omega_\mathrm{e}\left|\hat{i}_\mathrm{\beta}\right|^{1/2}$

两边同时约掉$\omega{e}$可以看出$\lambda$的值可以提前确认,有疑惑的点在于这里电流怎么处理?是一个计算值还是实时值.
个人推测可以推出来
$k_1>\frac{\omega _e\psi _f}{L_d}$

