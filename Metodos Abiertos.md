import tkinter as tk
from tkinter import messagebox, ttk
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg

# Función para evaluar f(x)
def f(x, expression):
    try:
        return eval(expression, {"x": x, "np": np})
    except Exception as e:
        messagebox.showerror("Error", f"Error en la función: {e}")
        return None

# Función para evaluar f'(x) (derivada)
def df(x, expression):
    h = 1e-6
    return (f(x + h, expression) - f(x, expression)) / h

# Método de Newton-Raphson
def newton_method():
    try:
        expression = func_entry.get()
        x0 = float(x0_entry.get())
        tol = float(threshold_entry.get())
        max_iter = int(iter_entry.get())

        iterations, values = [], []
        table.delete(*table.get_children())  # Limpiar tabla

        for i in range(max_iter):
            f_x0 = f(x0, expression)
            df_x0 = df(x0, expression)

            if abs(df_x0) < 1e-10:
                messagebox.showwarning("Advertencia", "Derivada cercana a cero, el método no puede continuar.")
                return

            x1 = x0 - f_x0 / df_x0
            f_x1 = f(x1, expression)

            table.insert("", "end", values=(i, f"{x1:.8f}", f"{f_x1:.8f}"))
            iterations.append(i)
            values.append(abs(f_x1))

            if abs(x1 - x0) < tol:
                messagebox.showinfo("Éxito", f"Raíz encontrada en x = {x1:.8f} en {i+1} iteraciones.")
                break

            x0 = x1  # Actualizar x0

        ax.clear()
        ax.plot(iterations, values, marker='o', linestyle='-')
        ax.set_xlabel("Iteraciones")
        ax.set_ylabel("|f(x)|")
        ax.set_title("Convergencia - Newton-Raphson")
        canvas.draw()

    except ValueError:
        messagebox.showerror("Error", "Ingrese valores numéricos válidos.")

# Método de la Secante
def secant_method():
    try:
        expression = func_entry.get()
        x0 = float(x0_entry.get())
        x1 = float(x1_entry.get())
        tol = float(threshold_entry.get())
        max_iter = int(iter_entry.get())

        iterations, values = [], []
        table.delete(*table.get_children())  # Limpiar tabla

        for i in range(max_iter):
            f_x0 = f(x0, expression)
            f_x1 = f(x1, expression)

            if abs(f_x1 - f_x0) < 1e-10:
                messagebox.showwarning("Advertencia", "División por cero detectada, el método no puede continuar.")
                return

            x2 = x1 - f_x1 * (x1 - x0) / (f_x1 - f_x0)
            f_x2 = f(x2, expression)

            table.insert("", "end", values=(i, f"{x2:.8f}", f"{f_x2:.8f}"))
            iterations.append(i)
            values.append(abs(f_x2))

            if abs(x2 - x1) < tol:
                messagebox.showinfo("Éxito", f"Raíz encontrada en x = {x2:.8f} en {i+1} iteraciones.")
                break

            x0, x1 = x1, x2  # Actualizar valores

        ax.clear()
        ax.plot(iterations, values, marker='o', linestyle='-')
        ax.set_xlabel("Iteraciones")
        ax.set_ylabel("|f(x)|")
        ax.set_title("Convergencia - Método de la Secante")
        canvas.draw()

    except ValueError:
        messagebox.showerror("Error", "Ingrese valores numéricos válidos.")

# Crear ventana principal
root = tk.Tk()
root.title("Métodos Numéricos: Newton-Raphson y Secante")
root.geometry("650x700")

# Entradas
frame_input = tk.Frame(root, padx=10, pady=10)
frame_input.pack()

tk.Label(frame_input, text="Función f(x):").grid(row=0, column=0, sticky=tk.W)
func_entry = tk.Entry(frame_input, width=30)
func_entry.grid(row=0, column=1)
func_entry.insert(0, "x**2 - 2")

tk.Label(frame_input, text="x0:").grid(row=1, column=0, sticky=tk.W)
x0_entry = tk.Entry(frame_input, width=10)
x0_entry.grid(row=1, column=1)

tk.Label(frame_input, text="x1 (Solo Secante):").grid(row=2, column=0, sticky=tk.W)
x1_entry = tk.Entry(frame_input, width=10)
x1_entry.grid(row=2, column=1)

tk.Label(frame_input, text="Tolerancia:").grid(row=3, column=0, sticky=tk.W)
threshold_entry = tk.Entry(frame_input, width=10)
threshold_entry.grid(row=3, column=1)
threshold_entry.insert(0, "1e-6")

tk.Label(frame_input, text="Iteraciones máx.:").grid(row=4, column=0, sticky=tk.W)
iter_entry = tk.Entry(frame_input, width=10)
iter_entry.grid(row=4, column=1)
iter_entry.insert(0, "50")

# Botones para calcular
frame_buttons = tk.Frame(root)
frame_buttons.pack()

btn_newton = tk.Button(frame_buttons, text="Método de Newton", command=newton_method, bg="lightblue")
btn_newton.grid(row=0, column=0, padx=5, pady=5)

btn_secant = tk.Button(frame_buttons, text="Método de la Secante", command=secant_method, bg="lightgreen")
btn_secant.grid(row=0, column=1, padx=5, pady=5)

# Tabla de resultados
frame_results = tk.Frame(root)
frame_results.pack()

tk.Label(frame_results, text="Resultados", font=("Arial", 12, "bold")).pack()
table = ttk.Treeview(frame_results, columns=("Iteración", "x", "f(x)"), show="headings", height=8)
table.heading("Iteración", text="Iteración")
table.heading("x", text="x")
table.heading("f(x)", text="f(x)")
table.pack()

# Gráfico de convergencia
frame_graph = tk.Frame(root)
frame_graph.pack()

fig, ax = plt.subplots(figsize=(5, 3))
canvas = FigureCanvasTkAgg(fig, master=frame_graph)
canvas.get_tk_widget().pack()

root.mainloop()
