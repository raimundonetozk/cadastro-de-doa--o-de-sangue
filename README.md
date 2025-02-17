# cadastro-de-doação-de-sangue

import tkinter as tk
from tkinter import messagebox
from tkinter import ttk
from tkcalendar import DateEntry
from datetime import datetime

# Guardar dados de doadores
doadores = {}

localidades = {
    "SP": ["São Paulo", "Campinas"],
    "RN": ["Natal", "Caicó"],
    "PB": ["Santa Luzia", "João Pessoa", "Patos"],
}

tipos_sanguineos = ["A+", "A-", "B+", "B-", "AB+", "AB-", "O+", "O-"]

# Atualizar lista de cidades
def atualizar_cidades(event, combobox_cidade):
    estado_selecionado = event.widget.get()
    if estado_selecionado in localidades:
        combobox_cidade['values'] = localidades[estado_selecionado]
    else:
        combobox_cidade['values'] = []

# Função para habilitar ou desabilitar campos do destinatário
def configurar_campos_destinatario(habilitar):
    estado = "normal" if habilitar else "disabled"
    entry_nome_destinatario.config(state=estado)
    entry_cpf_destinatario.config(state=estado)
    if not habilitar:
        entry_nome_destinatario.delete(0, tk.END)
        entry_cpf_destinatario.delete(0, tk.END)

# Função para cadastrar um novo doador
def cadastrar_doador():
    nome = entry_nome.get()
    tipo_sanguineo = combobox_tipo_sanguineo.get()
    ultima_doacao = date_entry_ultima_doacao.get()
    estado = combobox_estado.get()
    cidade = combobox_cidade.get()
    tem_destinatario = var_tem_destinatario.get()
    nome_destinatario = entry_nome_destinatario.get()
    cpf_destinatario = entry_cpf_destinatario.get()

    if nome and tipo_sanguineo and ultima_doacao and estado and cidade:
        if tem_destinatario and (not nome_destinatario or not cpf_destinatario):
            messagebox.showwarning("Atenção", "Informe o nome e CPF do destinatário!")
            return

        doadores[nome] = {
            'tipo_sanguineo': tipo_sanguineo,
            'ultima_doacao': ultima_doacao,
            'localizacao': f"{cidade}, {estado}",
            'tem_destinatario': tem_destinatario,
            'nome_destinatario': nome_destinatario,
            'cpf_destinatario': cpf_destinatario
        }
        messagebox.showinfo("Cadastro", f"Doador {nome} cadastrado com sucesso!")
        entry_nome.delete(0, tk.END)
        combobox_tipo_sanguineo.set('')
        date_entry_ultima_doacao.set_date(datetime.now())
        combobox_estado.set('')
        combobox_cidade.set('')
        var_tem_destinatario.set(False)
        configurar_campos_destinatario(False)
    else:
        messagebox.showwarning("Atenção", "Preencha todos os campos!")

# Enviar alerta caso exista doadores disponiveis
def enviar_alerta():
    tipo_sanguineo = combobox_alerta_tipo_sanguineo.get()
    estado = combobox_alerta_estado.get()
    cidade = combobox_alerta_cidade.get()
    localizacao_alerta = f"{cidade}, {estado}"
    
    doadores_compativeis = [
        nome for nome, dados in doadores.items()
        if dados['tipo_sanguineo'] == tipo_sanguineo and dados['localizacao'] == localizacao_alerta and not dados['tem_destinatario']
    ]

    if doadores_compativeis:
        print('Olá, tudo bem, precisamos de doadores com seu tipo sanguineo.')
        messagebox.showinfo("Alerta", f"Alerta enviado para doadores compatíveis!")
    else:
        messagebox.showwarning("Alerta", "Nenhum doador compatível encontrado.")
    
    combobox_alerta_tipo_sanguineo.set('')
    combobox_alerta_estado.set('')
    combobox_alerta_cidade.set('')

# Tkinter Module/interface
root = tk.Tk()
root.title("Sistema de Doação de Sangue")

# Cadastro de doadores
frame_cadastro = tk.Frame(root)
frame_cadastro.pack(padx=10, pady=10)

label_nome = tk.Label(frame_cadastro, text="Nome:")
label_nome.grid(row=0, column=0, sticky="e")
entry_nome = tk.Entry(frame_cadastro)
entry_nome.grid(row=0, column=1)

label_tipo_sanguineo = tk.Label(frame_cadastro, text="Tipo Sanguíneo:")
label_tipo_sanguineo.grid(row=1, column=0, sticky="e")
combobox_tipo_sanguineo = ttk.Combobox(frame_cadastro, values=tipos_sanguineos, state="readonly")
combobox_tipo_sanguineo.grid(row=1, column=1)

label_ultima_doacao = tk.Label(frame_cadastro, text="Última Doação:")
label_ultima_doacao.grid(row=2, column=0, sticky="e")
date_entry_ultima_doacao = DateEntry(frame_cadastro, date_pattern='dd/mm/yyyy')
date_entry_ultima_doacao.grid(row=2, column=1)

label_estado = tk.Label(frame_cadastro, text="Estado:")
label_estado.grid(row=3, column=0, sticky="e")
combobox_estado = ttk.Combobox(frame_cadastro, values=list(localidades.keys()), state="readonly")
combobox_estado.grid(row=3, column=1)
combobox_estado.bind("<<ComboboxSelected>>", lambda e: atualizar_cidades(e, combobox_cidade))

label_cidade = tk.Label(frame_cadastro, text="Cidade:")
label_cidade.grid(row=4, column=0, sticky="e")
combobox_cidade = ttk.Combobox(frame_cadastro, state="readonly")
combobox_cidade.grid(row=4, column=1)

var_tem_destinatario = tk.BooleanVar()
check_tem_destinatario = tk.Checkbutton(frame_cadastro, text="Tem Destinatário", variable=var_tem_destinatario, command=lambda: configurar_campos_destinatario(var_tem_destinatario.get()))
check_tem_destinatario.grid(row=5, columnspan=2, pady=5)

label_nome_destinatario = tk.Label(frame_cadastro, text="Nome:")
label_nome_destinatario.grid(row=6, column=0, sticky="e")
entry_nome_destinatario = tk.Entry(frame_cadastro, state="disabled")
entry_nome_destinatario.grid(row=6, column=1)

label_cpf_destinatario = tk.Label(frame_cadastro, text="CPF:")
label_cpf_destinatario.grid(row=7, column=0, sticky="e")
entry_cpf_destinatario = tk.Entry(frame_cadastro, state="disabled")
entry_cpf_destinatario.grid(row=7, column=1)

btn_cadastrar = tk.Button(frame_cadastro, text="Cadastrar Doador", command=cadastrar_doador)
btn_cadastrar.grid(row=8, columnspan=2, pady=10)


# Alerta para solicitar doadores

frame_alerta = tk.Frame(root)
frame_alerta.pack(padx=10, pady=10)

label_alerta_tipo_sanguineo = tk.Label(frame_alerta, text="Tipo Sanguíneo:")
label_alerta_tipo_sanguineo.grid(row=0, column=0, sticky="e")
combobox_alerta_tipo_sanguineo = ttk.Combobox(frame_alerta, values=tipos_sanguineos, state="readonly")
combobox_alerta_tipo_sanguineo.grid(row=0, column=1)

label_alerta_estado = tk.Label(frame_alerta, text="Estado:")
label_alerta_estado.grid(row=1, column=0, sticky="e")
combobox_alerta_estado = ttk.Combobox(frame_alerta, values=list(localidades.keys()), state="readonly")
combobox_alerta_estado.grid(row=1, column=1)
combobox_alerta_estado.bind("<<ComboboxSelected>>", lambda e: atualizar_cidades(e, combobox_alerta_cidade))

label_alerta_cidade = tk.Label(frame_alerta, text="Cidade:")
label_alerta_cidade.grid(row=2, column=0, sticky="e")
combobox_alerta_cidade = ttk.Combobox(frame_alerta, state="readonly")
combobox_alerta_cidade.grid(row=2, column=1)

btn_alerta = tk.Button(frame_alerta, text="Enviar Alerta", command=enviar_alerta)
btn_alerta.grid(row=3, columnspan=2, pady=10)

root.mainloop()
