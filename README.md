***Trabalho de extensão Universitário
Projeto de desenvolvimento de Sistema de Gerenciamento para controle de loja.
Estrutura Geral do Sistema
O sistema será dividido nas seguintes partes:
***
***
Clientes: Gerenciamento de clientes.
Fornecedores: Gerenciamento de fornecedores.
Estoque: Controle de produtos e inventário.
Vendas/Movimentações: Registro e controle de vendas e caixa.
Relatórios: Geração de relatórios de clientes, fornecedores, estoque e vendas em PDF e CSV.

Linguagem Python, interface gráfica PySimpleGUI e bibliotecas pandas e reporlab.
***
#Script SQL para criar as tabelas no PostgreSQL

CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cpf VARCHAR(11) UNIQUE NOT NULL,
    email VARCHAR(100)
);

CREATE TABLE fornecedores (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cnpj VARCHAR(14) UNIQUE NOT NULL,
    telefone VARCHAR(15),
    email VARCHAR(100)
);

CREATE TABLE produtos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco DECIMAL(10, 2) NOT NULL,
    quantidade INTEGER NOT NULL,
    fornecedor_id INTEGER REFERENCES fornecedores(id)
);

CREATE TABLE vendas (
    id SERIAL PRIMARY KEY,
    cliente_id INTEGER REFERENCES clientes(id),
    produto_id INTEGER REFERENCES produtos(id),
    quantidade INTEGER NOT NULL,
    valor_total DECIMAL(10, 2) NOT NULL,
    data_venda TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

#Classe de conexão com o Banco de Dados

import psycopg2

class Database:
    def __init__(self):
        self.conn = psycopg2.connect(
            dbname="sistema_loja",
            user="seu_usuario",
            password="sua_senha",
            host="localhost"
        )
        self.cursor = self.conn.cursor()

    def executar(self, query, params=None):
        self.cursor.execute(query, params)
        self.conn.commit()

    def fetch(self, query, params=None):
        self.cursor.execute(query, params)
        return self.cursor.fetchall()

    def fechar_conexao(self):
        self.cursor.close()
        self.conn.close()

#Interface para Gerenciamento de Clientes

import PySimpleGUI as sg

class InterfaceClientes:
    def __init__(self, db):
        self.db = db

    def tela_clientes(self):
        layout = [
            [sg.Text("Nome"), sg.Input(key="nome")],
            [sg.Text("CPF"), sg.Input(key="cpf")],
            [sg.Text("Email"), sg.Input(key="email")],
            [sg.Button("Adicionar Cliente"), sg.Button("Listar Clientes")],
            [sg.Output(size=(50, 10))]
        ]

        window = sg.Window("Gerenciamento de Clientes", layout)

        while True:
            event, values = window.read()

            if event == sg.WIN_CLOSED:
                break

            if event == "Adicionar Cliente":
                nome = values["nome"]
                cpf = values["cpf"]
                email = values["email"]
                if nome and cpf:
                    query = "INSERT INTO clientes (nome, cpf, email) VALUES (%s, %s, %s)"
                    self.db.executar(query, (nome, cpf, email))
                    print(f"Cliente {nome} adicionado com sucesso!")
                else:
                    print("Por favor, preencha todos os campos.")

            if event == "Listar Clientes":
                query = "SELECT * FROM clientes"
                clientes = self.db.fetch(query)
                for cliente in clientes:
                    print(f"ID: {cliente[0]}, Nome: {cliente[1]}, CPF: {cliente[2]}, Email: {cliente[3]}")

        window.close()

#Interface para Gerenciamento de Fornecedores

class InterfaceFornecedores:
    def __init__(self, db):
        self.db = db

    def tela_fornecedores(self):
        layout = [
            [sg.Text("Nome"), sg.Input(key="nome")],
            [sg.Text("CNPJ"), sg.Input(key="cnpj")],
            [sg.Text("Telefone"), sg.Input(key="telefone")],
            [sg.Text("Email"), sg.Input(key="email")],
            [sg.Button("Adicionar Fornecedor"), sg.Button("Listar Fornecedores")],
            [sg.Output(size=(50, 10))]
        ]

        window = sg.Window("Gerenciamento de Fornecedores", layout)

        while True:
            event, values = window.read()

            if event == sg.WIN_CLOSED:
                break

            if event == "Adicionar Fornecedor":
                nome = values["nome"]
                cnpj = values["cnpj"]
                telefone = values["telefone"]
                email = values["email"]
                if nome and cnpj:
                    query = "INSERT INTO fornecedores (nome, cnpj, telefone, email) VALUES (%s, %s, %s, %s)"
                    self.db.executar(query, (nome, cnpj, telefone, email))
                    print(f"Fornecedor {nome} adicionado com sucesso!")
                else:
                    print("Por favor, preencha todos os campos.")

            if event == "Listar Fornecedores":
                query = "SELECT * FROM fornecedores"
                fornecedores = self.db.fetch(query)
                for fornecedor in fornecedores:
                    print(f"ID: {fornecedor[0]}, Nome: {fornecedor[1]}, CNPJ: {fornecedor[2]}, Telefone: {fornecedor[3]}, Email: {fornecedor[4]}")

        window.close()

# Interface para Controle de Estoque

class InterfaceEstoque:
    def __init__(self, db):
        self.db = db

    def tela_estoque(self):
        layout = [
            [sg.Text("Nome do Produto"), sg.Input(key="nome_produto")],
            [sg.Text("Preço"), sg.Input(key="preco")],
            [sg.Text("Quantidade"), sg.Input(key="quantidade")],
            [sg.Text("ID Fornecedor"), sg.Input(key="fornecedor_id")],
            [sg.Button("Adicionar Produto"), sg.Button("Listar Estoque")],
            [sg.Output(size=(50, 10))]
        ]

        window = sg.Window("Gerenciamento de Estoque", layout)

        while True:
            event, values = window.read()

            if event == sg.WIN_CLOSED:
                break

            if event == "Adicionar Produto":
                nome = values["nome_produto"]
                preco = values["preco"]
                quantidade = values["quantidade"]
                fornecedor_id = values["fornecedor_id"]
                if nome and preco and quantidade and fornecedor_id:
                    query = "INSERT INTO produtos (nome, preco, quantidade, fornecedor_id) VALUES (%s, %s, %s, %s)"
                    self.db.executar(query, (nome, float(preco), int(quantidade), fornecedor_id))
                    print(f"Produto {nome} adicionado ao estoque com sucesso!")
                else:
                    print("Por favor, preencha todos os campos.")

            if event == "Listar Estoque":
                query = "SELECT * FROM produtos"
                produtos = self.db.fetch(query)
                for produto in produtos:
                    print(f"ID: {produto[0]}, Nome: {produto[1]}, Preço: {produto[2]}, Quantidade: {produto[3]}, Fornecedor ID: {produto[4]}")

        window.close()

#Interface para Controle de vendas

class InterfaceVendas:
    def __init__(self, db):
        self.db = db

    def tela_vendas(self):
        layout = [
            [sg.Text("ID do Cliente"), sg.Input(key="cliente_id")],
            [sg.Text("ID do Produto"), sg.Input(key="produto_id")],
            [sg.Text("Quantidade"), sg.Input(key="quantidade_venda")],
            [sg.Button("Registrar Venda"), sg.Button("Listar Vendas")],
            [sg.Output(size=(50, 10))]
        ]

        window = sg.Window("Gerenciamento de Vendas", layout)

        while True:
            event, values = window.read()

            if event == sg.WIN_CLOSED:
                break

            if event == "Registrar Venda":
                cliente_id = values["cliente_id"]
                produto_id = values["produto_id"]
                quantidade = values["quantidade_venda"]

                # Buscando o produto para verificar a quantidade disponível
                query = "SELECT preco, quantidade FROM produtos WHERE id = %s"
                produto = self.db.fetch(query, (produto_id,))
                if produto and int(quantidade) <= produto[0][1]:
                    preco = produto[0][0]
                    valor_total = float(preco) * int(quantidade)
                    
                    # Inserir venda
                    query_venda = "INSERT INTO vendas (cliente_id, produto_id, quantidade, valor_total) VALUES (%s, %s, %s, %s)"
                    self.db.executar(query_venda, (cliente_id, produto_id, quantidade, valor_total))
                    
                    # Atualizar o estoque
                    nova_quantidade = produto[0][1] - int(quantidade)
                    query_estoque = "UPDATE produtos SET quantidade = %s WHERE id = %s"
                    self.db.executar(query_estoque, (nova_quantidade, produto_id))
                    print(f"Venda registrada com sucesso. Valor total: R$ {valor_total:.2f}")
                else:
                    print("Produto indisponível ou quantidade solicitada acima do disponível.")
            
            if event == "Listar Vendas":
                query = "SELECT * FROM vendas"
                vendas = self.db.fetch(query)
                for venda in vendas:
                    print(f"ID: {venda[0]}, Cliente ID: {venda[1]}, Produto ID: {venda[2]}, Quantidade: {venda[3]}, Valor Total: {venda[4]}, Data: {venda[5]}")

        window.close()
        
#Relatórios em CSV

import pandas as pd

class RelatorioCSV:
    def __init__(self, db):
        self.db = db

    def gerar_relatorio_clientes(self):
        query = "SELECT * FROM clientes"
        clientes = self.db.fetch(query)
        df = pd.DataFrame(clientes, columns=["ID", "Nome", "CPF", "Email"])
        df.to_csv("relatorio_clientes.csv", index=False)
        print("Relatório de clientes gerado em CSV.")

    def gerar_relatorio_fornecedores(self):
        query = "SELECT * FROM fornecedores"
        fornecedores = self.db.fetch(query)
        df = pd.DataFrame(fornecedores, columns=["ID", "Nome", "CNPJ", "Telefone", "Email"])
        df.to_csv("relatorio_fornecedores.csv", index=False)
        print("Relatório de fornecedores gerado em CSV.")

    def gerar_relatorio_estoque(self):
        query = "SELECT * FROM produtos"
        produtos = self.db.fetch(query)
        df = pd.DataFrame(produtos, columns=["ID", "Nome", "Preço", "Quantidade", "Fornecedor ID"])
        df.to_csv("relatorio_estoque.csv", index=False)
        print("Relatório de estoque gerado em CSV.")

#Relatórios em PDF

from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas

class RelatorioPDF:
    def __init__(self, db):
        self.db = db

    def gerar_relatorio_vendas(self):
        query = "SELECT * FROM vendas"
        vendas = self.db.fetch(query)

        c = canvas.Canvas("relatorio_vendas.pdf", pagesize=letter)
        c.drawString(100, 750, "Relatório de Vendas")
        y = 730

        for venda in vendas:
            c.drawString(100, y, f"ID: {venda[0]}, Cliente ID: {venda[1]}, Produto ID: {venda[2]}, Quantidade: {venda[3]}, Valor Total: {venda[4]}")
            y -= 20

        c.save()
        print("Relatório de vendas gerado em PDF.")

# Execução do sistema completo

#Arquivo completo para integrar todas as funcionalidades

if __name__ == "__main__":
    db = Database()

    while True:
        layout = [
            [sg.Button("Gerenciar Clientes")],
            [sg.Button("Gerenciar Fornecedores")],
            [sg.Button("Gerenciar Estoque")],
            [sg.Button("Gerenciar Vendas")],
            [sg.Button("Gerar Relatórios")],
            [sg.Button("Sair")]
        ]

        window = sg.Window("Sistema de Gerenciamento de Loja", layout)
        event, _ = window.read()
        window.close()

        if event == sg.WIN_CLOSED or event == "Sair":
            break

        if event == "Gerenciar Clientes":
            InterfaceClientes(db).tela_clientes()

        if event == "Gerenciar Fornecedores":
            InterfaceFornecedores(db).tela_fornecedores()

        if event == "Gerenciar Estoque":
            InterfaceEstoque(db).tela_estoque()

        if event == "Gerenciar Vendas":
            InterfaceVendas(db).tela_vendas()

        if event == "Gerar Relatórios":
            layout_relatorios = [
                [sg.Button("Relatório de Clientes (CSV)")],
                [sg.Button("Relatório de Fornecedores (CSV)")],
                [sg.Button("Relatório de Estoque (CSV)")],
                [sg.Button("Relatório de Vendas (PDF)")],
                [sg.Button("Voltar")]
            ]
            window = sg.Window("Geração de Relatórios", layout_relatorios)
            event_relatorio, _ = window.read()
            window.close()

            if event_relatorio == "Relatório de Clientes (CSV)":
                RelatorioCSV(db).gerar_relatorio_clientes()

            if event_relatorio == "Relatório de Fornecedores (CSV)":
                RelatorioCSV(db).gerar_relatorio_fornecedores()

            if event_relatorio == "Relatório de Estoque (CSV)":
                RelatorioCSV(db).gerar_relatorio_estoque()

            if event_relatorio == "Relatório de Vendas (PDF)":
                RelatorioPDF(db).gerar_relatorio_vendas()

    db.fechar_conexao()

