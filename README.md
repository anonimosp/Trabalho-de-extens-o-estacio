# Trabalho-de-extens-o-estacio
Extensão Estacio

import tkinter as tk
from tkinter import ttk, messagebox
import pandas as pd
import datetime

class Item:
    def __init__(self, nome, quantidade_inicial, preco):
        self.nome = nome
        self.quantidade = quantidade_inicial
        self.preco = preco
        self.entradas = []
        self.saidas = []

    def adicionar_entrada(self, quantidade, data):
        self.quantidade += quantidade
        self.entradas.append((quantidade, data))

    def remover_saida(self, quantidade, data):
        if self.quantidade >= quantidade:
            self.quantidade -= quantidade
            self.saidas.append((quantidade, data))
        else:
            raise ValueError("Estoque insuficiente.")

    def valor_final(self):
        return self.quantidade * self.preco

    def get_historico(self):
        return self.entradas, self.saidas

class EstoqueApp:
    def __init__(self, master):
        self.master = master
        master.title("Sistema de Controle de Estoque")

        self.estoque = {}

        # Frame para adicionar item
        self.frame_adicionar = tk.Frame(master)
        self.frame_adicionar.pack(pady=10)

        tk.Label(self.frame_adicionar, text="Nome do Item:").grid(row=0, column=0)
        self.entry_nome = tk.Entry(self.frame_adicionar)
        self.entry_nome.grid(row=0, column=1)

        tk.Label(self.frame_adicionar, text="Quantidade Inicial:").grid(row=1, column=0)
        self.entry_quantidade = tk.Entry(self.frame_adicionar)
        self.entry_quantidade.grid(row=1, column=1)

        tk.Label(self.frame_adicionar, text="Preço Unitário:").grid(row=2, column=0)
        self.entry_preco = tk.Entry(self.frame_adicionar)
        self.entry_preco.grid(row=2, column=1)

        tk.Button(self.frame_adicionar, text="Adicionar Item", command=self.adicionar_item).grid(row=3, column=0, columnspan=2)

        # Frame para registrar entrada
        self.frame_entrada = tk.Frame(master)
        self.frame_entrada.pack(pady=10)

        tk.Label(self.frame_entrada, text="Nome do Item:").grid(row=0, column=0)
        self.entry_nome_entrada = tk.Entry(self.frame_entrada)
        self.entry_nome_entrada.grid(row=0, column=1)

        tk.Label(self.frame_entrada, text="Quantidade de Entrada:").grid(row=1, column=0)
        self.entry_quantidade_entrada = tk.Entry(self.frame_entrada)
        self.entry_quantidade_entrada.grid(row=1, column=1)

        tk.Button(self.frame_entrada, text="Registrar Entrada", command=self.registrar_entrada).grid(row=2, column=0, columnspan=2)

        # Frame para registrar saída
        self.frame_saida = tk.Frame(master)
        self.frame_saida.pack(pady=10)

        tk.Label(self.frame_saida, text="Nome do Item:").grid(row=0, column=0)
        self.entry_nome_saida = tk.Entry(self.frame_saida)
        self.entry_nome_saida.grid(row=0, column=1)

        tk.Label(self.frame_saida, text="Quantidade de Saída:").grid(row=1, column=0)
        self.entry_quantidade_saida = tk.Entry(self.frame_saida)
        self.entry_quantidade_saida.grid(row=1, column=1)

        tk.Button(self.frame_saida, text="Registrar Saída", command=self.registrar_saida).grid(row=2, column=0, columnspan=2)

        # Frame para consultar estoque
        self.frame_consultar = tk.Frame(master)
        self.frame_consultar.pack(pady=10)

        self.tree = ttk.Treeview(self.frame_consultar, columns=("Nome", "Quantidade", "Preço", "Valor Final"), show="headings")
        self.tree.heading("Nome", text="Nome")
        self.tree.heading("Quantidade", text="Quantidade")
        self.tree.heading("Preço", text="Preço")
        self.tree.heading("Valor Final", text="Valor Final")
        self.tree.pack()

        tk.Button(self.frame_consultar, text="Atualizar Estoque", command=self.atualizar_estoque).pack()
        tk.Button(self.frame_consultar, text="Gerar Relatório Excel", command=self.gerar_relatorio).pack()

    def adicionar_item(self):
        nome = self.entry_nome.get()
        quantidade = self.entry_quantidade.get()
        preco = self.entry_preco.get()

        if self.validar_dados(nome, quantidade, preco):
            self.estoque[nome] = Item(nome, int(quantidade), float(preco))
            self.limpar_campos(self.frame_adicionar)
            self.atualizar_estoque()
            messagebox.showinfo("Sucesso", f"Item {nome} adicionado ao estoque.")

    def registrar_entrada(self):
        nome = self.entry_nome_entrada.get()
        quantidade = self.entry_quantidade_entrada.get()

        if self.validar_dados(nome, quantidade):
            if nome in self.estoque:
                data = datetime.date.today().strftime("%Y-%m-%d")
                self.estoque[nome].adicionar_entrada(int(quantidade), data)
                self.limpar_campos(self.frame_entrada)
                self.atualizar_estoque()
                messagebox.showinfo("Sucesso", f"Entrada registrada para {nome}.")
            else:
                messagebox.showerror("Erro", f"Item {nome} não encontrado no estoque.")

    def registrar_saida(self):
        nome = self.entry_nome_saida.get()
        quantidade = self.entry_quantidade_saida.get()

        if self.validar_dados(nome, quantidade):
            if nome in self.estoque:
                data = datetime.date.today().strftime("%Y-%m-%d")
                try:
                    self.estoque[nome].remover_saida(int(quantidade), data)
                    self.limpar_campos(self.frame_saida)
                    self.atualizar_estoque()
                    messagebox.showinfo("Sucesso", f"Saída registrada para {nome}.")
                except ValueError as e:
                    messagebox.showerror("Erro", str(e))
            else:
                messagebox.showerror("Erro", f"Item {nome} não encontrado no estoque.")

    def atualizar_estoque(self):
        for item in self.tree.get_children():
            self.tree.delete(item)

        for nome, item in self.estoque.items():
            self.tree.insert("", "end", values=(nome, item.quantidade, item.preco, item.valor_final()))

    def gerar_relatorio(self):
        data = []
        for nome, item in self.estoque.items():
            data.append({
                "Nome": nome,
                "Quantidade": item.quantidade,
                "Preço": item.preco,
                "Valor Final": item.valor_final()
            })
        df = pd.DataFrame(data)
        df.to_excel("relatorio_estoque.xlsx", index=False)
        messagebox.showinfo("Sucesso", "Relatório gerado com sucesso!")

    def validar_dados(self, *args):
        for arg in args:
            if not arg or not isinstance(arg, str):
                return False
            if any(not c.isdigit() for c in arg if c != "."):
                return False
        return True

    def limpar_campos(self, frame):
        for widget in frame.winfo_children():
            if isinstance(widget, tk.Entry):
                widget.delete(0, tk.END)

root = tk.Tk()
app = EstoqueApp(root)
root.mainloop()
