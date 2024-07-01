#AI #Python
[[AI提示词]]
你是一个专业的编程设计师，写Python代码，在终端内运行，实现向用户询问问题，记录数据生成表格。
首先你需要先创建一个Excel表格
以下是需要问的问题：
第一个问题**请输入学校名称和学校代号，名称和代号用空格隔开：**。第二个问题**请输入(此处应为前一个输入的学校名称)的专业组代号：**。第三个问题**请输入专业代号，每个代号间用空格隔开，最大填入六组数字(若不满六组数字，后面空缺的位置则不填写)，例如'001 002'，请填写并回车提交：**。
问完这三个问题之后，请提示已写入文件，再回到第一个问题以此类推，在第一个问题给出退出的选项，以终止这个循环。
通过询问得到的问题，生成Excel表格，关于表格部分表头第一行，应依次为**学校名称，学校代号，专业组代号，专业代号1，专业代号2，专业代号3，专业代号4，专业代号5，专业代号6**。下方的数据则来自于向用户询问得到的数据。
Excel表格的生成位置应该在运行该py程序的根目录重命名为**大学志愿填报**

```python
import tkinter as tk
from tkinter import messagebox
from tkinter import ttk
import openpyxl
from openpyxl import Workbook
import os

# 定义文件名
filename = "大学志愿填报.xlsx"

# 创建Excel工作簿和工作表
if not os.path.exists(filename):
    wb = Workbook()
    ws = wb.active
    ws.title = "填报数据"
    # 设置表头
    headers = ["学校名称", "学校代号", "专业组代号", "专业代号1", "专业代号2", "专业代号3", "专业代号4", "专业代号5", "专业代号6"]
    ws.append(headers)
    wb.save(filename)

# 定义撤销操作的全局变量
last_entry = None

# 加载Excel数据
def load_data():
    wb = openpyxl.load_workbook(filename)
    ws = wb.active
    data = []
    for row in ws.iter_rows(min_row=2, values_only=True):
        data.append(row)
    return data

# 定义GUI界面
def main():
    def submit_data():
        global last_entry
        
        school_name = school_name_entry.get()
        school_code = school_code_entry.get()
        major_group_code = major_group_code_entry.get()
        major_codes = major_codes_entry.get().split()

        if len(school_name) == 0 or len(school_code) == 0 or len(major_group_code) == 0 or len(major_codes) == 0:
            messagebox.showwarning("输入错误", "所有字段都必须填写！")
            return

        if len(major_codes) > 6:
            messagebox.showwarning("输入错误", "专业代号不能超过六个！")
            return

        # 补全至六个专业代号
        while len(major_codes) < 6:
            major_codes.append("")

        # 记录数据
        data = [school_name, school_code, major_group_code] + major_codes

        # 打开Excel文件并写入数据
        wb = openpyxl.load_workbook(filename)
        ws = wb.active
        ws.append(data)
        wb.save(filename)

        last_entry = data

        # 清空输入框
        school_name_entry.delete(0, tk.END)
        school_code_entry.delete(0, tk.END)
        major_group_code_entry.delete(0, tk.END)
        major_codes_entry.delete(0, tk.END)

        messagebox.showinfo("成功", "数据已写入文件")
        update_treeview()

    def undo_last_entry():
        global last_entry
        
        if not last_entry:
            messagebox.showwarning("撤销错误", "没有操作可以撤销")
            return

        # 打开Excel文件并删除最后一行数据
        wb = openpyxl.load_workbook(filename)
        ws = wb.active

        # 查找并删除最后一行数据
        for row in ws.iter_rows(min_row=2, max_row=ws.max_row):
            if [cell.value for cell in row] == last_entry:
                ws.delete_rows(row[0].row, 1)
                break

        wb.save(filename)
        last_entry = None
        messagebox.showinfo("成功", "已撤销上一次操作")
        update_treeview()

    def update_treeview():
        # 清空Treeview
        for item in tree.get_children():
            tree.delete(item)
        
        # 重新加载数据
        data = load_data()
        for row in data:
            tree.insert("", tk.END, values=row)

    def delete_selected():
        selected_item = tree.selection()
        if not selected_item:
            messagebox.showwarning("删除错误", "请选择要删除的项目")
            return

        item = tree.item(selected_item)
        values = item['values']

        # 打开Excel文件并删除选定数据
        wb = openpyxl.load_workbook(filename)
        ws = wb.active

        # 查找并删除选定数据
        for row in ws.iter_rows(min_row=2, max_row=ws.max_row):
            if [cell.value for cell in row] == values:
                ws.delete_rows(row[0].row, 1)
                break

        wb.save(filename)
        messagebox.showinfo("成功", "已删除选定项目")
        update_treeview()

    def on_enter_pressed(event, next_entry):
        next_entry.focus_set()

    # 创建主窗口
    root = tk.Tk()
    root.title("大学志愿填报")
    root.geometry("1024x768")

    # 创建输入区
    input_frame = tk.Frame(root, pady=20)
    input_frame.pack(pady=20)

    tk.Label(input_frame, text="学校名称:", font=("Arial", 12)).grid(row=0, column=0, padx=10, pady=5)
    school_name_entry = tk.Entry(input_frame, font=("Arial", 12), width=30)
    school_name_entry.grid(row=0, column=1, padx=10, pady=5)
    school_name_entry.bind("<Return>", lambda event: on_enter_pressed(event, school_code_entry))

    tk.Label(input_frame, text="学校代号:", font=("Arial", 12)).grid(row=1, column=0, padx=10, pady=5)
    school_code_entry = tk.Entry(input_frame, font=("Arial", 12), width=30)
    school_code_entry.grid(row=1, column=1, padx=10, pady=5)
    school_code_entry.bind("<Return>", lambda event: on_enter_pressed(event, major_group_code_entry))

    tk.Label(input_frame, text="专业组代号:", font=("Arial", 12)).grid(row=2, column=0, padx=10, pady=5)
    major_group_code_entry = tk.Entry(input_frame, font=("Arial", 12), width=30)
    major_group_code_entry.grid(row=2, column=1, padx=10, pady=5)
    major_group_code_entry.bind("<Return>", lambda event: on_enter_pressed(event, major_codes_entry))

    tk.Label(input_frame, text="专业代号（用空格隔开，最多六个）:", font=("Arial", 12)).grid(row=3, column=0, padx=10, pady=5)
    major_codes_entry = tk.Entry(input_frame, font=("Arial", 12), width=30)
    major_codes_entry.grid(row=3, column=1, padx=10, pady=5)
    major_codes_entry.bind("<Return>", lambda event: on_enter_pressed(event, submit_button))

    # 创建提交和撤销按钮
    button_frame = tk.Frame(root, pady=10)
    button_frame.pack(pady=10)

    submit_button = tk.Button(button_frame, text="提交", command=submit_data, font=("Arial", 12), bg="#4CAF50", fg="white", relief="groove")
    submit_button.grid(row=0, column=0, padx=10)

    undo_button = tk.Button(button_frame, text="撤销", command=undo_last_entry, font=("Arial", 12), bg="#F44336", fg="white", relief="groove")
    undo_button.grid(row=0, column=1, padx=10)

    delete_button = tk.Button(button_frame, text="删除选定项目", command=delete_selected, font=("Arial", 12), bg="#FFC107", fg="black", relief="groove")
    delete_button.grid(row=0, column=2, padx=10)

    # 创建Treeview来显示数据
    tree_frame = tk.Frame(root)
    tree_frame.pack(pady=20)

    columns = ["学校名称", "学校代号", "专业组代号", "专业代号1", "专业代号2", "专业代号3", "专业代号4", "专业代号5", "专业代号6"]
    tree = ttk.Treeview(tree_frame, columns=columns, show="headings")
    for col in columns:
        tree.heading(col, text=col)
        tree.column(col, width=100)
    
    tree.pack(side="left")

    scrollbar = ttk.Scrollbar(tree_frame, orient="horizontal", command=tree.xview)
    tree.configure(xscrollcommand=scrollbar.set)
    scrollbar.pack(side="bottom", fill="x")

    # 初始化Treeview数据
    update_treeview()

    # 运行主循环
    root.mainloop()

if __name__ == "__main__":
    main()
```
