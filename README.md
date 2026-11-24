"""
Projeto: Proteção de informações - criptografia e hash com Python

O programa tem três partes:
1. Cifra de César (substituição com deslocamento fixo)
2. Cifra de substituição com uma chave (alfabeto embaralhado)
3. Hash SHA-256 (verificar integridade de texto e guardar senhas com sal)
"""

import hashlib
import os
import string

ALFABETO = string.ascii_lowercase

# Chave da cifra de substituição: cada letra do alfabeto é trocada
# pela letra que está na mesma posição aqui (precisa ter as 26 letras).
CHAVE = "qwertyuiopasdfghjklzxcvbnm"


# ---------------------------------------------------------------
# 1) Cifra de César
# ---------------------------------------------------------------
def cifrar_cesar(texto, deslocamento):
    """Desloca cada letra 'deslocamento' posições no alfabeto."""
    resultado = ""
    for caractere in texto:
        if caractere.lower() in ALFABETO:
            posicao = ALFABETO.index(caractere.lower())
            nova_letra = ALFABETO[(posicao + deslocamento) % 26]
            # Mantém maiúscula/minúscula do texto original
            resultado += nova_letra.upper() if caractere.isupper() else nova_letra
        else:
            # Espaços, números e pontuação ficam como estão
            resultado += caractere
    return resultado


def decifrar_cesar(texto, deslocamento):
    """Desfaz a cifra: é só deslocar para o lado contrário."""
    return cifrar_cesar(texto, -deslocamento)


# ---------------------------------------------------------------
# 2) Cifra de substituição com chave
# ---------------------------------------------------------------
def codificar_substituicao(texto, chave=CHAVE):
    """Troca cada letra do alfabeto pela letra correspondente da chave."""
    tabela = str.maketrans(ALFABETO + ALFABETO.upper(), chave + chave.upper())
    return texto.translate(tabela)


def decodificar_substituicao(texto, chave=CHAVE):
    """Faz o caminho inverso: da letra da chave de volta para a original."""
    tabela = str.maketrans(chave + chave.upper(), ALFABETO + ALFABETO.upper())
    return texto.translate(tabela)


# ---------------------------------------------------------------
# 3) Hash
# ---------------------------------------------------------------
def gerar_hash(texto):
    """Gera o hash SHA-256 de um texto (sempre o mesmo para o mesmo texto)."""
    return hashlib.sha256(texto.encode("utf-8")).hexdigest()


def verificar_integridade(texto, hash_esperado):
    """Confere se o texto continua igual ao original comparando os hashes."""
    return gerar_hash(texto) == hash_esperado


def hash_de_senha(senha):
    """Guarda a senha com um 'sal' aleatório, nunca a senha pura."""
    sal = os.urandom(16).hex()
    hash_salvo = hashlib.sha256((sal + senha).encode("utf-8")).hexdigest()
    return sal, hash_salvo


def verificar_senha(senha, sal, hash_salvo):
    """Recalcula o hash com o mesmo sal e compara com o que foi guardado."""
    return hashlib.sha256((sal + senha).encode("utf-8")).hexdigest() == hash_salvo


# ---------------------------------------------------------------
# Menu
# ---------------------------------------------------------------
def ler_deslocamento():
    while True:
        try:
            return int(input("Deslocamento (número inteiro, ex: 3): "))
        except ValueError:
            print("Digite um número inteiro.")


def menu():
    # Cadastro de senha usado nas opções 6 e 7
    senha_guardada = None

    while True:
        print("\n=== Proteção de informações ===")
        print("1 - Codificar com cifra de César")
        print("2 - Decodificar com cifra de César")
        print("3 - Codificar com cifra de substituição")
        print("4 - Decodificar com cifra de substituição")
        print("5 - Gerar hash SHA-256 de um texto")
        print("6 - Cadastrar senha (guarda só o hash com sal)")
        print("7 - Verificar senha")
        print("0 - Sair")

        opcao = input("Escolha uma opção: ").strip()

        if opcao == "1":
            texto = input("Mensagem: ")
            print("Codificada:", cifrar_cesar(texto, ler_deslocamento()))
        elif opcao == "2":
            texto = input("Mensagem codificada: ")
            print("Decodificada:", decifrar_cesar(texto, ler_deslocamento()))
        elif opcao == "3":
            texto = input("Mensagem: ")
            print("Codificada:", codificar_substituicao(texto))
        elif opcao == "4":
            texto = input("Mensagem codificada: ")
            print("Decodificada:", decodificar_substituicao(texto))
        elif opcao == "5":
            texto = input("Texto: ")
            print("SHA-256:", gerar_hash(texto))
        elif opcao == "6":
            senha = input("Nova senha: ")
            senha_guardada = hash_de_senha(senha)
            print("Senha cadastrada. Hash guardado:", senha_guardada[1])
        elif opcao == "7":
            if senha_guardada is None:
                print("Cadastre uma senha primeiro (opção 6).")
            else:
                tentativa = input("Digite a senha: ")
                if verificar_senha(tentativa, *senha_guardada):
                    print("Senha correta!")
                else:
                    print("Senha incorreta.")
        elif opcao == "0":
            print("Até mais!")
            break
        else:
            print("Opção inválida.")


if __name__ == "__main__":
    menu()
