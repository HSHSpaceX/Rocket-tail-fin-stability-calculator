# Rocket-tail-fin-stability-calculator火箭尾翼稳定性计算器
<img width="1914" height="1080" alt="image" src="https://github.com/user-attachments/assets/b64fd1bd-cd17-426e-8445-f383953d1ff0" />
大家在使用Openrocket绘制火箭的基本模型图时，常常会不断调整尾翼的形状，计算出适合自己火箭稳定性的尾翼，但这种方法十分麻烦，并且Openrocket不支持直接导出pxf格式文件用于3D打印模型。因此我用Python开发语言为大家制作了一个火箭尾翼稳定计算工具，该软件支持火箭的重量分布提供尾翼安装位置等提示建议，并且会通过绘制发射抛物线来简单模拟不同尾翼对火箭飞行轨迹的影响，以及偏离垂直发射高度的度数，并且该图纸支持PNG格式导出，这是该软件的基本信息。软件界面使用T K库GUI页面导出。下面是软件安装包打开即可使用。

# python源代码
 
  
  import tkinter as tk from tkinter import ttk, messagebox, filedialog
  import numpy as np
  import matplotlib.pyplot as plt from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
  import matplotlib
  import dxfwrite from dxfwrite import DXFEngine as dxf

  matplotlib.rcParams['font.sans-serif'] = ['SimHei']  # 用来正常显示中文标签
  matplotlib.rcParams['axes.unicode_minus'] = False    # 用来正常显示负号

  class RocketStabilityCalculator:
    def __init__(self, root):
        self.root = root
        self.root.title("火箭尾翼稳定性计算器HSH编写")
        self.root.geometry("1200x800")
        
        # 输入变量
        self.length_var = tk.DoubleVar(value=500.0)  # 厘米
        self.front_weight_var = tk.DoubleVar(value=10000.0)  # 克
        self.mid_weight_var = tk.DoubleVar(value=20000.0)    # 克
        self.rear_weight_var = tk.DoubleVar(value=15000.0)   # 克
        self.wind_speed_var = tk.DoubleVar(value=5.0)
        self.launch_angle_var = tk.DoubleVar(value=45.0)
        self.target_altitude_var = tk.DoubleVar(value=1000.0)
        self.wing_shape_var = tk.StringVar(value="梯形")
        self.wing_count_var = tk.IntVar(value=4)
        self.root_chord_var = tk.DoubleVar(value=30.0)  # 厘米
        self.tip_chord_var = tk.DoubleVar(value=15.0)   # 厘米
        self.wing_span_var = tk.DoubleVar(value=40.0)   # 厘米
        self.wing_position_var = tk.DoubleVar(value=0.8)
        
        # 创建主框架
        self.main_frame = ttk.Frame(root)
        self.main_frame.pack(fill=tk.BOTH, expand=True)
        
        # 输入区域
        self.create_input_section()
        
        # 结果区域
        self.create_output_section()
        
        # 绘图区域
        self.create_plot_section()
        
        # 底部按钮
        self.create_button_section()
    
    def create_input_section(self):
        input_frame = ttk.LabelFrame(self.main_frame, text="火箭参数")
        input_frame.grid(row=0, column=0, padx=10, pady=10, sticky="nsew")
        
        # 火箭基本参数
        ttk.Label(input_frame, text="火箭总长 (cm):").grid(row=0, column=0, sticky="w")
        ttk.Entry(input_frame, textvariable=self.length_var).grid(row=0, column=1)
        
        ttk.Label(input_frame, text="前段重量 (g):").grid(row=1, column=0, sticky="w")
        ttk.Entry(input_frame, textvariable=self.front_weight_var).grid(row=1, column=1)
        
        ttk.Label(input_frame, text="中段重量 (g):").grid(row=2, column=0, sticky="w")
        ttk.Entry(input_frame, textvariable=self.mid_weight_var).grid(row=2, column=1)
        
        ttk.Label(input_frame, text="后段重量 (g):").grid(row=3, column=0, sticky="w")
        ttk.Entry(input_frame, textvariable=self.rear_weight_var).grid(row=3, column=1)
        
        # 环境参数
        ttk.Label(input_frame, text="风速 (m/s):").grid(row=4, column=0, sticky="w")
        ttk.Entry(input_frame, textvariable=self.wind_speed_var).grid(row=4, column=1)
        
        ttk.Label(input_frame, text="发射角度 (度):").grid(row=5, column=0, sticky="w")
        ttk.Entry(input_frame, textvariable=self.launch_angle_var).grid(row=5, column=1)
        
        ttk.Label(input_frame, text="目标高度 (m):").grid(row=6, column=0, sticky="w")
        ttk.Entry(input_frame, textvariable=self.target_altitude_var).grid(row=6, column=1)
        
        # 尾翼参数
        ttk.Label(input_frame, text="尾翼形状:").grid(row=7, column=0, sticky="w")
        shape_combobox = ttk.Combobox(input_frame, textvariable=self.wing_shape_var, 
                                    values=["矩形", "梯形", "三角形", "椭圆", "后掠"])
        shape_combobox.grid(row=7, column=1)
        
        ttk.Label(input_frame, text="尾翼数量:").grid(row=8, column=0, sticky="w")
        ttk.Entry(input_frame, textvariable=self.wing_count_var).grid(row=8, column=1)
        
        ttk.Label(input_frame, text="根部弦长 (cm):").grid(row=9, column=0, sticky="w")
        ttk.Entry(input_frame, textvariable=self.root_chord_var).grid(row=9, column=1)
        
        ttk.Label(input_frame, text="尖部弦长 (cm):").grid(row=10, column=0, sticky="w")
        ttk.Entry(input_frame, textvariable=self.tip_chord_var).grid(row=10, column=1)
        
        ttk.Label(input_frame, text="翼展 (cm):").grid(row=11, column=0, sticky="w")
        ttk.Entry(input_frame, textvariable=self.wing_span_var).grid(row=11, column=1)
        
        ttk.Label(input_frame, text="安装位置 (%):").grid(row=12, column=0, sticky="w")
        ttk.Scale(input_frame, from_=0, to=100, variable=self.wing_position_var, 
                 orient=tk.HORIZONTAL, length=150).grid(row=12, column=1)
        
        # 计算按钮
        ttk.Button(input_frame, text="计算稳定性", command=self.calculate_stability).grid(row=13, column=0, columnspan=2, pady=10)
    
    def create_output_section(self):
        output_frame = ttk.LabelFrame(self.main_frame, text="计算结果")
        output_frame.grid(row=0, column=1, padx=10, pady=10, sticky="nsew")
        
        # 结果文本区域
        self.result_text = tk.Text(output_frame, height=15, width=50)
        self.result_text.pack(fill=tk.BOTH, expand=True, padx=5, pady=5)
        self.result_text.insert(tk.END, "计算结果将显示在这里...")
        
        # 尾翼示意图
        self.wing_canvas = tk.Canvas(output_frame, width=200, height=150, bg="white")
        self.wing_canvas.pack(pady=10)
    
    def create_plot_section(self):
        plot_frame = ttk.LabelFrame(self.main_frame, text="飞行偏离角度")
        plot_frame.grid(row=1, column=0, columnspan=2, padx=10, pady=10, sticky="nsew")
        
        # 创建绘图区域
        self.fig, self.ax = plt.subplots(figsize=(8, 4))
        self.canvas = FigureCanvasTkAgg(self.fig, master=plot_frame)
        self.canvas.get_tk_widget().pack(fill=tk.BOTH, expand=True)
        
        # 初始绘图
        self.update_plot()
    
    def create_button_section(self):
        button_frame = ttk.Frame(self.main_frame)
        button_frame.grid(row=2, column=0, columnspan=2, pady=10)
        
        ttk.Button(button_frame, text="导出尾翼DXF", command=self.export_dxf).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="保存偏离图", command=self.save_plot).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="重置参数", command=self.reset_parameters).pack(side=tk.LEFT, padx=5)
        ttk.Button(button_frame, text="退出", command=self.root.quit).pack(side=tk.RIGHT, padx=5)
    
    def calculate_stability(self):
        try:
            # 获取输入值（转换为米）
            L = self.length_var.get() / 100  # 厘米转米
            w_front = self.front_weight_var.get() / 1000  # 克转千克
            w_mid = self.mid_weight_var.get() / 1000      # 克转千克
            w_rear = self.rear_weight_var.get() / 1000    # 克转千克
            wind_speed = self.wind_speed_var.get()
            launch_angle = self.launch_angle_var.get()
            
            # 计算重心位置
            total_weight = w_front + w_mid + w_rear
            cg = (w_front * 0.2 * L + w_mid * 0.5 * L + w_rear * 0.8 * L) / total_weight
            
            # 计算压力中心 (简化模型)
            cp = L * 0.65  # 简化计算
            
            # 计算稳定性裕度
            stability_margin = (cp - cg) / L * 100  # 百分比
            
            # 计算风偏角
            wind_angle = np.arctan(wind_speed / 10) * 180 / np.pi  # 简化模型
            
            # 计算尾翼参数
            wing_position = L * (self.wing_position_var.get() / 100)
            
            # 获取尾翼尺寸（厘米）
            root_chord_cm = self.root_chord_var.get()
            tip_chord_cm = self.tip_chord_var.get()
            wing_span_cm = self.wing_span_var.get()
            
            # 显示结果（使用厘米单位）
            self.result_text.delete(1.0, tk.END)
            self.result_text.insert(tk.END, f"=== 火箭稳定性报告 ===\n\n")
            self.result_text.insert(tk.END, f"重心位置: {cg*100:.1f} cm\n")
            self.result_text.insert(tk.END, f"压力中心: {cp*100:.1f} cm\n")
            self.result_text.insert(tk.END, f"稳定性裕度: {stability_margin:.1f}%\n")
            
            # 稳定性评估
            if stability_margin < 5:
                self.result_text.insert(tk.END, " 警告: 稳定性不足! 火箭可能失控\n")
            elif stability_margin < 10:
                self.result_text.insert(tk.END, " 注意: 稳定性较低\n")
            else:
                self.result_text.insert(tk.END, " 稳定性良好\n")
                
            self.result_text.insert(tk.END, f"推荐尾翼位置: {L*0.7*100:.1f} - {L*0.9*100:.1f} cm\n")
            self.result_text.insert(tk.END, f"风偏角: {wind_angle:.1f} 度\n")
            self.result_text.insert(tk.END, f"尾翼安装位置: {wing_position*100:.1f} cm\n")
            self.result_text.insert(tk.END, f"尾翼尺寸: 根弦 {root_chord_cm:.1f} cm, ")
            self.result_text.insert(tk.END, f"尖弦 {tip_chord_cm:.1f} cm, ")
            self.result_text.insert(tk.END, f"翼展 {wing_span_cm:.1f} cm\n")
            
            # 更新尾翼示意图
            self.draw_wing()
            
            # 更新偏离图
            self.update_plot(stability_margin)
            
        except Exception as e:
            messagebox.showerror("计算错误", f"发生错误: {str(e)}")
    
    def draw_wing(self):
        # 清除画布
        self.wing_canvas.delete("all")
        
        # 获取尾翼参数（厘米）
        root_chord = self.root_chord_var.get() * 1.5  # 缩放以便显示
        tip_chord = self.tip_chord_var.get() * 1.5
        wing_span = self.wing_span_var.get() * 1.0
        shape = self.wing_shape_var.get()
        
        # 中心位置
        center_x, center_y = 100, 75
        
        # 根据形状绘制
        if shape == "矩形":
            points = [
                center_x - root_chord/2, center_y - wing_span/2,
                center_x + root_chord/2, center_y - wing_span/2,
                center_x + root_chord/2, center_y + wing_span/2,
                center_x - root_chord/2, center_y + wing_span/2
            ]
            self.wing_canvas.create_polygon(points, fill="lightblue", outline="black")
        
        elif shape == "梯形":
            points = [
                center_x - root_chord/2, center_y - wing_span/2,
                center_x + root_chord/2, center_y - wing_span/2,
                center_x + tip_chord/2, center_y + wing_span/2,
                center_x - tip_chord/2, center_y + wing_span/2
            ]
            self.wing_canvas.create_polygon(points, fill="lightgreen", outline="black")
        
        elif shape == "三角形":
            points = [
                center_x, center_y - wing_span/2,
                center_x + root_chord/2, center_y + wing_span/2,
                center_x - root_chord/2, center_y + wing_span/2
            ]
            self.wing_canvas.create_polygon(points, fill="lightyellow", outline="black")
        
        elif shape == "椭圆":
            self.wing_canvas.create_oval(
                center_x - root_chord/2, center_y - wing_span/2,
                center_x + root_chord/2, center_y + wing_span/2,
                fill="lightpink", outline="black"
            )
        
        elif shape == "后掠":
            points = [
                center_x - root_chord/2, center_y - wing_span/2,
                center_x + root_chord/3, center_y - wing_span/2,
                center_x + tip_chord/2, center_y + wing_span/2,
                center_x - tip_chord/1.5, center_y + wing_span/2
            ]
            self.wing_canvas.create_polygon(points, fill="lightcyan", outline="black")
        
        # 添加标签
        self.wing_canvas.create_text(center_x, center_y - wing_span/2 - 15, 
                                   text=f"{self.wing_count_var.get()}片 {shape}尾翼")
    
    def update_plot(self, stability_margin=10.0):
        # 清除之前的绘图
        self.ax.clear()
        
        # 模拟轨迹数据
        launch_angle = np.radians(self.launch_angle_var.get())
        wind_speed = self.wind_speed_var.get()
        target_alt = self.target_altitude_var.get()
        
        # 简化的轨迹计算
        t = np.linspace(0, 100, 500)
        v0 = 150  # 初始速度 m/s
        g = 9.81  # 重力加速度
        
        # 考虑风偏的轨迹
        x = v0 * np.cos(launch_angle) * t + wind_speed * t**0.5
        y = v0 * np.sin(launch_angle) * t - 0.5 * g * t**2
        
        # 计算偏离角度（相对于垂直中心轴）
        # 避免除以零错误
        with np.errstate(divide='ignore', invalid='ignore'):
            deviation_angle = np.degrees(np.arctan(x / y))
        
        # 处理无效值（当y=0时）
        deviation_angle = np.nan_to_num(deviation_angle, nan=0.0, posinf=90.0, neginf=-90.0)
        
        # 找到最大高度点
        max_alt_idx = np.argmax(y)
        max_alt = y[max_alt_idx]
        max_dev_angle = deviation_angle[max_alt_idx]
        
        # 找到火箭落地时间点（高度<0）
        ground_idx = np.argmax(y < 0)
        if ground_idx == 0:  # 如果没有找到落地点
            ground_idx = len(y) - 1
        
        # 截取到落地前的数据
        t = t[:ground_idx]
        y = y[:ground_idx]
        deviation_angle = deviation_angle[:ground_idx]
        
        # 添加不稳定性影响
        stability_factor = max(0.1, min(1.0, stability_margin / 15.0))
        instability_factor = 1.0 / stability_factor
        
        # 当尾翼安装位置不当导致不稳定时，偏离角度会急剧增加
        deviation_angle = deviation_angle * instability_factor
        
        # 绘制偏离角度随高度的变化
        self.ax.plot(deviation_angle, y, 'b-', linewidth=2, label='飞行偏离角度')
        
        # 标记最大高度点
        if max_alt_idx < ground_idx:
            self.ax.plot(deviation_angle[max_alt_idx], max_alt, 'ro', 
                        label=f'最大高度: {max_alt:.0f}m\n偏离角度: {deviation_angle[max_alt_idx]:.1f}°')
        
        # 绘制目标高度线
        if max_alt > target_alt:
            self.ax.axhline(y=target_alt, color='g', linestyle='--', 
                           label=f'目标高度: {target_alt:.0f}m')
        else:
            self.ax.axhline(y=target_alt, color='r', linestyle='--', 
                           label=f'未达目标: {target_alt:.0f}m')
        
        # 绘制零米高度线
        self.ax.axhline(y=0, color='k', linestyle='-', linewidth=1, label='地面')
        
        # 添加不稳定警告
        if stability_margin < 5:
            self.ax.text(0.5, 0.5, "警告: 尾翼安装不当!\n火箭可能失控", 
                        transform=self.ax.transAxes, fontsize=14,
                        color='red', ha='center', va='center',
                        bbox=dict(facecolor='yellow', alpha=0.5))
        
        # 设置图形属性
        self.ax.set_title("火箭飞行偏离角度")
        self.ax.set_xlabel("偏离垂直中心轴角度 (°)")
        self.ax.set_ylabel("高度 (m)")
        self.ax.grid(True, linestyle='--', alpha=0.7)
        self.ax.legend()
        self.ax.set_ylim(0, max(1500, max_alt * 1.2))
        
        # 重绘画布
        self.canvas.draw()
    
    def export_dxf(self):
        file_path = filedialog.asksaveasfilename(
            defaultextension=".dxf",
            filetypes=[("DXF 文件", "*.dxf"), ("所有文件", "*.*")]
        )
        
        if not file_path:
            return
        
        try:
            # 创建DXF文件
            dwg = dxf.drawing(file_path)
            
            # 获取尾翼参数（厘米）
            shape = self.wing_shape_var.get()
            count = self.wing_count_var.get()
            root_chord = self.root_chord_var.get()
            tip_chord = self.tip_chord_var.get()
            wing_span = self.wing_span_var.get()
            
            # 添加尾翼形状（使用厘米单位）
            if shape == "矩形":
                points = [
                    (0, 0),
                    (root_chord, 0),
                    (root_chord, wing_span),
                    (0, wing_span),
                    (0, 0)
                ]
                dwg.add(dxf.polyline(points, layer='WING'))
            
            elif shape == "梯形":
                points = [
                    (0, 0),
                    (root_chord, 0),
                    (root_chord - (root_chord - tip_chord)/2, wing_span),
                    ((root_chord - tip_chord)/2, wing_span),
                    (0, 0)
                ]
                dwg.add(dxf.polyline(points, layer='WING'))
            
            elif shape == "三角形":
                points = [
                    (root_chord/2, 0),
                    (root_chord, wing_span),
                    (0, wing_span),
                    (root_chord/2, 0)
                ]
                dwg.add(dxf.polyline(points, layer='WING'))
            
            # 保存文件
            dwg.save()
            messagebox.showinfo("导出成功", f"尾翼设计已保存为 DXF 文件:\n{file_path}")
            
        except Exception as e:
            messagebox.showerror("导出错误", f"无法保存 DXF 文件: {str(e)}")
    
    def save_plot(self):
        file_path = filedialog.asksaveasfilename(
            defaultextension=".png",
            filetypes=[("PNG 图像", "*.png"), ("JPEG 图像", "*.jpg"), ("所有文件", "*.*")]
        )
        
        if not file_path:
            return
        
        try:
            self.fig.savefig(file_path, dpi=300)
            messagebox.showinfo("保存成功", f"偏离角度图已保存为:\n{file_path}")
        except Exception as e:
            messagebox.showerror("保存错误", f"无法保存图像: {str(e)}")
    
    def reset_parameters(self):
        # 重置所有输入变量
        self.length_var.set(500.0)  # 厘米
        self.front_weight_var.set(10000.0)  # 克
        self.mid_weight_var.set(20000.0)    # 克
        self.rear_weight_var.set(15000.0)   # 克
        self.wind_speed_var.set(5.0)
        self.launch_angle_var.set(45.0)
        self.target_altitude_var.set(1000.0)
        self.wing_shape_var.set("梯形")
        self.wing_count_var.set(4)
        self.root_chord_var.set(30.0)  # 厘米
        self.tip_chord_var.set(15.0)   # 厘米
        self.wing_span_var.set(40.0)   # 厘米
        self.wing_position_var.set(80.0)
        
        # 清除结果
        self.result_text.delete(1.0, tk.END)
        self.result_text.insert(tk.END, "参数已重置，请重新计算...")
        
        # 重绘图
        self.draw_wing()
        self.update_plot()

 #主程序
if __name__ == "__main__":
    root = tk.Tk()
    app = RocketStabilityCalculator(root)
    root.mainloop()>

# 使用教程
1.填写参数
<img width="1920" height="1009" alt="image" src="https://github.com/user-attachments/assets/4b068a39-33fe-49fa-b6df-050d42211715" />
2. 显示计算结果、尾翼样式、火箭飞行偏离角度图
![image](https://github.com/user-attachments/assets/50da609b-42bd-40bc-9c5b-658c2d9d433e)
3.可选择导出尾翼DXF格式文件等功能
<img width="472" height="101" alt="image" src="https://github.com/user-attachments/assets/8a717915-a401-461c-a2f6-4e07eec1ea2f" />
# 文章链接
https://www.kechuang.org/t/91289
