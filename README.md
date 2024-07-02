import os

def limpar_tela():
    os.system('cls' if os.name == 'nt' else 'clear')

def mostrar_boas_vindas():
    limpar_tela()
    print("""
    =======================================
               Bem-vindo ao Dio Bank
    =======================================
    """)
    input("Aperte qualquer tecla para continuar...")

Dio_Bank = "Dio Bank" 

mostrar_boas_vindas()
limpar_tela()

print(f"Bem vindo ao {Dio_Bank}!")

Menu = """
[c] Cadastrar Usuário
[a] Abrir Conta
[l] Listar Usuários
[u] Listar Contas
[f] Fechar Conta
[d] Depositar
[s] Sacar
[e] Extrato
[q] Sair

=> """

usuarios = []
contas = []
LIMITE_SAQUES = 3
AGENCIA = "0001"

def confirmar_operacao(mensagem):
    while True:
        confirmacao = input(f"{mensagem} (S/N): ").upper()
        if confirmacao in ("S", "N"):
            return confirmacao == "S"
        else:
            print("Opção inválida. Digite S ou N.")

def cadastrar_usuario(*args, **kwargs):
    cpf = kwargs.get('cpf') or input("Informe o CPF (somente números): ").replace('.', '').replace('-', '')
    usuario_existente = next((usuario for usuario in usuarios if usuario["cpf"] == cpf), None)

    if usuario_existente:
        print("Já existe um usuário com esse CPF!")
        return

    nome = kwargs.get('nome') or input("Informe o nome completo: ")
    endereco = kwargs.get('endereco') or input("Informe o endereço (Formato: Rua, Número, Bairro, Cidade/Estado): ")

    user_id = len(usuarios) + 1
    usuarios.append({"id": user_id, "nome": nome, "cpf": cpf, "endereco": endereco})
    print("Usuário cadastrado com sucesso!")

def abrir_conta(*args, **kwargs):
    cpf = kwargs.get('cpf') or input("Informe o CPF do usuário: ").replace('.', '').replace('-', '')
    usuario = next((usuario for usuario in usuarios if usuario["cpf"] == cpf), None)

    if not usuario:
        print("Usuário não encontrado! Cadastre o usuário primeiro.")
        return

    numero_conta = len(contas) + 1
    contas.append({"user_id": usuario["id"], "numero_conta": numero_conta, "agencia": AGENCIA, "saldo": 0, "extrato": "", "numero_saques": 0})
    print(f"Conta {numero_conta} - Agência {AGENCIA} aberta com sucesso para {usuario['nome']}!")

def listar_usuarios():
    if not usuarios:
        print("Nenhum usuário cadastrado.")
    else:
        print("\nUsuários cadastrados:")
        for usuario in usuarios:
            print(f"ID: {usuario['id']}, Nome: {usuario['nome']}, CPF: {usuario['cpf']}, Endereço: {usuario['endereco']}")

def listar_contas():
    if not contas:
        print("Nenhuma conta cadastrada.")
    else:
        print("\nContas cadastradas:")
        for conta in contas:
            usuario = next((usuario for usuario in usuarios if usuario["id"] == conta["user_id"]), None)
            print(f"Conta: {conta['numero_conta']}, Agência: {conta['agencia']}, Titular: {usuario['nome']}, Saldo: R$ {conta['saldo']:.2f}")

def obter_conta_usuario(*args, **kwargs):
    cpf = kwargs.get('cpf') or input("Informe o CPF do usuário: ").replace('.', '').replace('-', '')
    usuario = next((usuario for usuario in usuarios if usuario["cpf"] == cpf), None)
    
    if not usuario:
        print("Usuário não encontrado!")
        return None

    contas_usuario = [conta for conta in contas if conta["user_id"] == usuario["id"]]
    if not contas_usuario:
        print("Nenhuma conta encontrada para este usuário.")
        return None

    numero_conta = kwargs.get('numero_conta') or int(input("Informe o número da conta: "))
    agencia = kwargs.get('agencia') or input("Informe a agência: ")
    conta = next((conta for conta in contas_usuario if conta["numero_conta"] == numero_conta and conta["agencia"] == agencia), None)
    if not conta:
        print("Conta não encontrada.")
        return None

    return conta

def depositar(*args, **kwargs):
    conta = kwargs.get('conta')
    valor = kwargs.get('valor') or float(input("Informe o valor do depósito: "))
    if not conta:
        conta = obter_conta_usuario()
        if not conta:
            return

    if valor <= 0:
        raise ValueError("Valor do depósito deve ser positivo.")

    if confirmar_operacao(f"Confirmar depósito de R$ {valor:.2f}?"):
        conta["saldo"] += valor
        conta["extrato"] += f"Depósito: R$ {valor:.2f}\n"
        print("Depósito realizado com sucesso!")
    else:
        print("Depósito cancelado.")

def sacar(*args, **kwargs):
    conta = kwargs.get('conta')
    valor = kwargs.get('valor') or float(input("Informe o valor do saque: "))
    if not conta:
        conta = obter_conta_usuario()
        if not conta:
            return

    if valor <= 0:
        raise ValueError("Valor do saque deve ser positivo.")

    if valor > conta["saldo"]:
        raise ValueError("Saldo insuficiente para saque.")

    if valor > 500:
        raise ValueError("Valor do saque excede o limite de R$ 500.")

    if conta["numero_saques"] >= LIMITE_SAQUES:
        raise ValueError("Número máximo de saques diários excedido.")

    if confirmar_operacao(f"Confirmar saque de R$ {valor:.2f}?"):
        conta["saldo"] -= valor
        conta["extrato"] += f"Saque: R$ {valor:.2f}\n"
        conta["numero_saques"] += 1
        print("Saque realizado com sucesso!")
    else:
        print("Saque cancelado.")

def exibir_extrato(*args, **kwargs):
    conta = kwargs.get('conta')
    if not conta:
        conta = obter_conta_usuario()
        if not conta:
            return

    print("\n================ EXTRATO ================\n")
    print("Não foram realizadas movimentações." if not conta["extrato"] else conta["extrato"])
    print(f"\nSaldo: R$ {conta['saldo']:.2f}")
    print("==========================================")

def fechar_conta(*args, **kwargs):
    conta = kwargs.get('conta')
    if not conta:
        conta = obter_conta_usuario()
        if not conta:
            return

    if confirmar_operacao(f"Tem certeza que deseja fechar a conta {conta['numero_conta']} da agência {conta['agencia']}?"):
        contas.remove(conta)
        print("Conta fechada com sucesso!")
    else:
        print("Operação de fechamento de conta cancelada.")

while True:
    opcao = input(Menu)

    if opcao == "c":
        cadastrar_usuario()

    elif opcao == "a":
        abrir_conta()

    elif opcao == "l":
        listar_usuarios()

    elif opcao == "u":
        listar_contas()

    elif opcao == "f":
        fechar_conta()

    elif opcao == "d":
        depositar()

    elif opcao == "s":
        sacar()

    elif opcao == "e":
        exibir_extrato()

    elif opcao == "q":
        print("Obrigado por usar o Dio Bank!")
        break

    else:
        print("Operação inválida, por favor selecione novamente a operação desejada.")
