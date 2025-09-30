# -Modelando_o_Sistema_Banc-rio_em_POO_-com_Python

from abc import ABC
from datetime import datetime
import textwrap

def menu():
    menu = """\
    ================= Menu ==================
    [1]\tDepositar
    [2]\tSacar
    [3]\tExtrato
    [4]\tNovo usuário
    [5]\tNova conta
    [6]\tListar contas
    [7]\tCadastrar Chave PIX
    [8]\tSair
    => """
    return input(textwrap.dedent(menu))

class Cliente:
    def __init__(self, endereco):
        self.endereco = endereco
        self.conta = []
    
    def realizar_transacao(self, conta, transacao):
        transacao.registrar(conta)
    
    def adicionar_conta(self, conta):
        self.conta.append(conta)   
                                  
class PessoaFisica(Cliente):
    def __init__(self, nome, data_nascimento, cpf, endereco):
        super().__init__(endereco)
        self.nome = nome
        self.data_nascimento = data_nascimento
        self.cpf = cpf
    
class Conta:
    def __init__(self, numero, cliente):
        self._saldo = 0
        self._numero = numero
        self._agencia = "0001"
        self._cliente = cliente
        self._historico = Historico()
        self._chave_pix = None
        
    @classmethod
    def nova_conta(cls, cliente, numero):
        return cls(numero, cliente)    
    
    @property
    def saldo(self):
        return self._saldo

    @property
    def numero(self):
        return self._numero

    @property
    def agencia(self):
        return self._agencia

    @property
    def cliente(self):
        return self._cliente

    @property
    def historico(self):
        return self._historico

    def sacar(self, valor):
        saldo = self.saldo
        excedeu_saldo = valor > saldo
        
        if excedeu_saldo:
            print("\n@@@ Operação falhou! Você não tem saldo suficiente. @@@")

        elif valor > 0:
            self._saldo -= valor
            print("\n=== Saque realizado com sucesso! ===")
            return True

        else:
            print("\n@@@ Operação falhou! O valor informado é inválido. @@@")

        return False

    def depositar(self, valor):
        if valor > 0:
            self._saldo += valor
            print("\n=== Depósito realizado com sucesso! ===")
        else:
            print("\n@@@ Operação falhou! O valor informado é inválido. @@@")
            return False

        return True
    
    def pix(self, valor, conta_destino):
        if valor <= 0:
            print("\n@@@ Operação falhou! O valor informado é inválido. @@@")
            return False

        if self._saldo < valor:
            print("\n@@@ Operação falhou! Você não tem saldo suficiente. @@@")
            return False

        if not hasattr(conta_destino, 'chave_pix') or not conta_destino.chave_pix:
            print("\n@@@ Operação falhou! Conta destino não possui chave Pix cadastrada. @@@")
            return False

        self._saldo -= valor
        conta_destino._saldo += valor
        print(f"\n=== Transferência de R${valor:.2f} via Pix para {conta_destino.chave_pix} realizada com sucesso! ===")
        return True
   
class ContaCorrente(Conta):
    def __init__(self, numero, cliente, limite=500, limite_saques=3):
        super().__init__(numero, cliente)
        self.limite = limite
        self.limite_saques = limite_saques
        self.chave_pix = None  # atributo para armazenar a chave Pix

    def sacar(self, valor):
        numero_saques = len(
            [transacao for transacao in getattr(self.historico, "transacoes", []) if transacao.get("tipo") == "Saque"]
        )

        excedeu_limite = valor > self.limite
        excedeu_saques = numero_saques >= self.limite_saques

        if excedeu_limite:
            print("\n@@@ Operação falhou! O valor do saque excede o limite. @@@")

        elif excedeu_saques:
            print("\n@@@ Operação falhou! Número máximo de saques excedido. @@@")

        else:
            return super().sacar(valor)

        return False

    def cadastrar_pix(self, chave):
        self.chave_pix = chave

    def transferir_pix(self, valor, conta_destino):
        if hasattr(conta_destino, 'chave_pix') and conta_destino.chave_pix:
            print(f"Transferência de R${valor:.2f} via Pix para {conta_destino.chave_pix} realizada com sucesso.")
        else:
            print("Conta destino não possui chave Pix cadastrada.")

    def __str__(self):
        return f"""\
            Agência:\t{self.agencia}
            C/C:\t\t{self.numero}
            Titular:\t{self.cliente.nome}
        """

class ContaPoupanca(Conta):
    def __init__(self, numero, cliente, limite=500, limite_saques=3):
        super().__init__(numero, cliente)
        self.limite = limite
        self.limite_saques = limite_saques
        self.chave_pix = None  # atributo para armazenar a chave Pix

    def sacar(self, valor):
        numero_saques = len(
            [transacao for transacao in getattr(self.historico, "transacoes", []) if transacao.get("tipo") == "Saque"]
        )

        excedeu_limite = valor > self.limite
        excedeu_saques = numero_saques >= self.limite_saques

        if excedeu_limite:
            print("\n@@@ Operação falhou! O valor do saque excede o limite. @@@")

        elif excedeu_saques:
            print("\n@@@ Operação falhou! Número máximo de saques excedido. @@@")

        else:
            return super().sacar(valor)

        return False

    def cadastrar_pix(self, chave):
        self.chave_pix = chave

    def transferir_pix(self, valor, conta_destino):
        if hasattr(conta_destino, 'chave_pix') and conta_destino.chave_pix:
            print(f"Transferência de R${valor:.2f} via Pix para {conta_destino.chave_pix} realizada com sucesso.")
        else:
            print("Conta destino não possui chave Pix cadastrada.")

    def __str__(self):
        return f"""\
            Agência:\t{self.agencia}
            C/C:\t\t{self.numero}
            Titular:\t{self.cliente.nome}
        """

class Historico: 
    def __init__(self):
        self._transacoes = []

    @property
    def transacoes(self):
        return self._transacoes

    def adicionar_transacao(self, transacao):
        self._transacoes.append(
            {
                "tipo": transacao.__class__.__name__,
                "valor": transacao.valor,
                "data": datetime.now().strftime("%d-%m-%Y %H:%M:%S"),
            }    
        )
    
class Transacao(ABC):
    def __init__(self):
        super().__init__()
        self.valor = None

    def registrar(self, conta):
        pass

class Saque(Transacao):
    def __init__(self, valor):
        super().__init__()
        self._valor = valor

    @property
    def valor(self):
        return self._valor

    def registrar(self, conta):
        sucesso_transacao = conta.sacar(self.valor)
        if sucesso_transacao:
            conta.historico.adicionar_transacao(self)

class Deposito(Transacao):
    def __init__(self, valor):
        super().__init__()
        self._valor = valor

    @property
    def valor(self):
        return self._valor

    def registrar(self, conta):
        sucesso_transacao = conta.depositar(self.valor)
        if sucesso_transacao:
            conta.historico.adicionar_transacao(self)

def exibir_extrato(Clientes):
    raise NotImplementedError

def cadastrar_novo_usuario(Clientes):
    print("=== Cadastro de Novo Usuário ===")
    nome = input("Nome completo: ")
    data_nascimento = input("Data de nascimento (dd-mm-aaaa): ")
    cpf = input("CPF (somente números): ")
    endereco = input("Endereço (logradouro, número - bairro - cidade/sigla estado): ")

    # Verifica se já existe usuário com o mesmo CPF
    if any(c for c in Clientes if hasattr(c, "cpf") and c.cpf == cpf):
        print("\n@@@ Já existe um usuário com esse CPF! @@@")
    else:
        novo_usuario = PessoaFisica(nome, data_nascimento, cpf, endereco)
        Clientes.append(novo_usuario)
        print("\n=== Usuário cadastrado com sucesso! ===")

def criar_conta(numero_conta, Clientes, contas):
    print("Função de criação de conta ainda não implementada.")
    # Aqui você pode implementar a lógica de criação de conta futuramente

def lista_contas(contas):
    if not contas:
        print("Nenhuma conta cadastrada.")
    else:
        for conta in contas:
            print(conta)

def main():
    Clientes = []
    contas = []
    
    while True:
        opcao = menu()
        
        if opcao == "1": 
            # Exemplo de lógica para depósito
            cpf = input("Informe o CPF do cliente para depósito: ")
            cliente = next((c for c in Clientes if hasattr(c, "cpf") and c.cpf == cpf), None)
            if not cliente or not cliente.conta:
                print("Cliente não encontrado ou não possui conta.")
            else:
                conta = cliente.conta[0]  # Considerando a primeira conta do cliente
                valor = float(input("Informe o valor do depósito: "))
                deposito = Deposito(valor)
                deposito.registrar(conta)
        
        elif opcao == "2":
            cpf = input("Informe o CPF do cliente para saque: ")
            cliente = next((c for c in Clientes if hasattr(c, "cpf") and c.cpf == cpf), None)
            if not cliente or not cliente.conta:
                print("Cliente não encontrado ou não possui conta.")
            else:
                conta = cliente.conta[0]  # Considerando a primeira conta do cliente
                valor = float(input("Informe o valor do saque: "))
                saque = Saque(valor)
                saque.registrar(conta)
        
        elif opcao == "3":
            cpf = input("Informe o CPF do cliente para extrato: ")
            cliente = next((c for c in Clientes if hasattr(c, "cpf") and c.cpf == cpf), None)
            if not cliente or not cliente.conta:
                print("Cliente não encontrado ou não possui conta.")
            else:
                conta = cliente.conta[0]  # Considerando a primeira conta do cliente
                print("\n================ EXTRATO ================")
                transacoes = conta.historico.transacoes
                if not transacoes:
                    print("Não foram realizadas movimentações.")
                else:
                    for transacao in transacoes:
                        print(f"{transacao['data']} - {transacao['tipo']}: R$ {transacao['valor']:.2f}")
                print(f"\nSaldo atual: R$ {conta.saldo:.2f}")
                print("==========================================")
            
        elif opcao == "4":
            cadastrar_novo_usuario(Clientes)
        
        elif opcao == "5":
            numero_conta = len(contas) + 1
            criar_conta(numero_conta,Clientes, contas)
            
        elif opcao == "6":
            lista_contas(contas)
            
        elif opcao == "7":
            cpf = input("Informe o CPF do cliente para cadastrar chave Pix: ")
            cliente = next((c for c in Clientes if hasattr(c, "cpf") and c.cpf == cpf), None)
            if not cliente or not cliente.conta:
                print("Cliente não encontrado ou não possui conta.")
            else:
                conta = cliente.conta[0]  # Considerando a primeira conta do cliente
                chave = input("Informe a chave Pix a ser cadastrada: ")
                if hasattr(conta, "cadastrar_pix"):
                    conta.cadastrar_pix(chave)
                    print("Chave Pix cadastrada com sucesso!")
                else:
                    print("Esta conta não suporta cadastro de chave Pix.")
            
        elif opcao == "8":
            break    
        
        else:
            print("\n@@@ Operação invalida, por favor selecione novamente a operação desejada. @@@") 
            
main()            
