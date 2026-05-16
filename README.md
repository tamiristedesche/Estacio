"""
==================================================================
SISTEMA DE REGISTROS DE ALUNOS - CRUD COM SQLITE
==================================================================
Disciplina: Desenvolvimento Rápido de Aplicações em Python
Alunos: Tamiris da Silva Tedesche Cerdeira - matrícula: 202508191831
        Samuel de Holanda Reis             -            202504275037

Funcionalidades: Cadastrar, Listar, Buscar, Atualizar e Excluir
Banco de Dados: SQLite (arquivo: escola.db)
Interface: Texto (terminal)
==================================================================
"""
# ================================================================
# BIBLIOTECAS
# ================================================================
import sqlite3 # Para trabalhar com o banco de dados SQLite
# ================================================================
# PARTE 1: FUNÇÕES DO BANCO DE DADOS
# ================================================================
def conectar():
    """
    FUNÇÃO: conectar()
    OBJETIVO: Abrir a conexão com o banco de dados
    RETORNA: Um objeto de conexão com o arquivo escola.db
    """
    return sqlite3.connect("escola.db")
def criar_tabela():
    """
    FUNÇÃO: criar_tabela()
    OBJETIVO: Criar a tabela 'alunos' se ela não existir
    OBSERVAÇÃO: Esta função é chamada apenas uma vez, no início do programa
    """
    # 1. Conectar ao banco
    conexao = conectar()
    cursor = conexao.cursor()
    # 2. Comando SQL para criar a tabela
    cursor.execute ("""
          CREATE TABLE IF NOT EXISTS alunos (
             id INTEGER PRIMARY KEY AUTOINCREMENT,
             nome TEXT NOT NULL,
             matricula TEXT UNIQUE NOT NULL, 
             curso TEXT NOT NULL,
             nota1 REAL,
             nota2 REAL              
          )  
    """) 
    # 3. Salvar e fechar
    conexao.commit()
    conexao.close()
# ================================================================
# PARTE 2: OPERAÇÕES CRUD (Create, Read, Update, Delete)
# ================================================================
# ----------------------------------------------------------------
# CREATE: Casdastrar novo aluno
# ----------------------------------------------------------------
def cadastrar_aluno(nome, matricula, curso, nota1, nota2):
    """
    FUNÇÃO: cadastrar_aluno()
    OBJETIVO: Inserir um novo aluno no banco de dados
    PARÂMETROS: nome, matricula, curso, nota1, nota2
    RETORNA: True (sucesso) ou False (erro, geralmente matrícula duplicada)
    """
    try:
        conexao = conectar()
        cursor = conexao.cursor()
        
        cursor.execute("""
            INSERT INTO alunos (nome, matricula, curso, nota1, nota2)
            VALUES (?, ?, ?, ?, ?)
        """, (nome, matricula, curso, nota1, nota2))
        
        conexao.commit()
        conexao.close()
        return True  # Cadastro realizado com sucesso
        
    except sqlite3.IntegrityError:
        # Este erro ocorre quando a matrícula já existe
        return False  # Falha no cadastro
    
# ----------------------------------------------------------------
# READ: Casdastrar novo aluno
# ----------------------------------------------------------------
def listar_alunos():
    """
    FUNÇÃO: listar_alunos()
    OBJETIVO: Buscar todos os alunos cadastrados
    RETORNA: Lista de tuplas [(id, nome, matricula, curso, nota1, nota2)]
    """
    conexao = conectar()
    cursor = conexao.cursor()
    cursor.execute(""" 
        SELECT id, nome, matricula, curso, nota1, nota2
        FROM alunos
        ORDER BY id
        """)
    alunos = cursor.fetchall()
    conexao.close()
    return alunos
# ----------------------------------------------------------------
# READ: Buscar aluno específico
# ----------------------------------------------------------------
def buscar_aluno(termo):
    """
    FUNÇÃO: buscar_aluno()
    OBJETIVO: Encontrar alunos pelo nome ou matrícula
    PARÂMETRO: termo - texto digitado pelo usuário
    RETORNA: Lista de alunos que correspodem à busca
    """
    conexao = conectar()
    cursor = conexao.cursor()
    # O símbolo % significa "qualquer caractere"
    # Exemplo: %joão% encontra "João", "Joãozinho", "Maria João"
    cursor.execute("""
        SELECT id, nome, matricula, curso, nota1, nota2
        FROM alunos
        WHERE nome LIKE ? OR matricula LIKE ?
        ORDER BY id
    """, (f"%{termo}%", f"%{termo}%"))
    
    resultados = cursor.fetchall()
    conexao.close()
    return resultados
# ----------------------------------------------------------------
# UPDATE: Atualizar dados de um aluno
# ----------------------------------------------------------------
def atualizar_aluno(id_aluno, nome, curso, nota1, nota2):
    """
    FUNÇÃO: atualizar_alunos
    OBJETIVO: Modificar os dados de um aluno existente
    PARÂMETROS: id_aluno (identificador), nome, curso, nota1, nota2
    """
    conexao = conectar()
    cursor = conexao.cursor()
    
    cursor.execute("""
        UPDATE alunos
        SET nome = ?, curso = ?, nota1 = ?, nota2 = ?
        WHERE id = ?
    """, (nome, curso, nota1, nota2, id_aluno))
    
    conexao.commit()
    conexao.close()
# ----------------------------------------------------------------
# DELETE: Excluir um aluno
# ----------------------------------------------------------------
def excluir_aluno(id_aluno):
    """
    FUNÇÃO: excluir_aluno()
    OBJETIVO: Remover um aluno do banco de dados
    PARÂMETRO: id_aluno (identificador do aluno a ser removido)
    """
    conexao = conectar()
    cursor = conexao.cursor()
    
    cursor.execute("DELETE FROM alunos WHERE id = ?", (id_aluno,))
    
    conexao.commit()
    conexao.close()
# ================================================================
# PARTE 3: INTERFACE COM O USUÁRIO (TUDO QUE APARECE NA TELA)
# ================================================================
def exibir_menu():
    """
    FUNÇÃO: exibir_menu()
    OBJETIVO: Mostrar o menu principal com todas as opções
    """
    print("\n" + "=" * 50)
    print("        SISTEMA DE REGISTRO DE ALUNOS")
    print("=" * 50)
    print("  1 → Cadastrar novo aluno")
    print("  2 → Listar todos os alunos")
    print("  3 → Buscar aluno por nome ou matrícula")
    print("  4 → Atualizar dados de um aluno")
    print("  5 → Excluir um aluno")
    print("  0 → Sair do sistema")
    print("-" * 50)
def mostrar_alunos(alunos, titulo="LISTA DE ALUNOS"):
    """
    FUNÇÃO: mostrar_alunos()
    OBJETIVO: Exibir os alunos de forma organizada na tela
    PARÂMETROS: 
        - alunos: lista de tuplas com os dados
        - titulo: título que aparece antes da lista
    """
    # Verifica se a lista está vazia
    if not alunos:
        print("\n⚠️  Nenhum aluno encontrado!")
        return
    
    # Mostra o título
    print(f"\n{'=' * 50}")
    print(f"  {titulo}")
    print(f"{'=' * 50}")
    
    # Para cada aluno, calcula média e situação
    for aluno in alunos:
        id_aluno, nome, matricula, curso, nota1, nota2 = aluno
        
        # Cálculo da média
        if nota1 is not None and nota2 is not None:
            media = (nota1 + nota2) / 2
        else:
            media = 0
        
        # Define situação (média >= 6 = APROVADO)
        situacao = "APROVADO ✅" if media >= 6 else "REPROVADO ❌"
        
        # Exibe os dados
        print(f"\n  📌 ID: {id_aluno}")
        print(f"     Nome: {nome}")
        print(f"     Matrícula: {matricula}")
        print(f"     Curso: {curso}")
        print(f"     Notas: {nota1:.1f} e {nota2:.1f}")
        print(f"     Média: {media:.1f} - {situacao}")
        print("-" * 50)
def cadastrar_interface():
    """
    FUNÇÃO: cadastrar_interface()
    OBJETIVO: Interface para o usuário cadastrar um novo aluno
    """
    print("\n" + "=" * 50)
    print("        CADASTRO DE NOVO ALUNO")
    print("=" * 50)
    
    # Coleta os dados do usuário
    nome = input("  Nome completo: ").strip()
    matricula = input("  Matrícula: ").strip()
    curso = input("  Curso: ").strip()
    
    # Tratamento de erro para notas
    try:
        nota1 = float(input("  Primeira nota: "))
        nota2 = float(input("  Segunda nota: "))
    except ValueError:
        print("\n  ❌ ERRO: Notas devem ser números!")
        return
    
    # Tenta cadastrar
    if cadastrar_aluno(nome, matricula, curso, nota1, nota2):
        print(f"\n  ✅ ALUNO CADASTRADO COM SUCESSO!")
    else:
        print(f"\n  ❌ ERRO: Matrícula {matricula} já existe!")
def listar_interface():
    """
    FUNÇÃO: listar_interface()
    OBJETIVO: Interface para listar todos os alunos
    """
    alunos = listar_alunos()
    mostrar_alunos(alunos, "TODOS OS ALUNOS CADASTRADOS")
def buscar_interface():
    """
    FUNÇÃO: buscar_interface()
    OBJETIVO: Interface para buscar alunos por nome ou matrícula
    """
    print("\n" + "=" * 50)
    print("        BUSCAR ALUNO")
    print("=" * 50)
    
    termo = input("  Digite o nome ou matrícula: ").strip()
    
    if not termo:
        print("\n  ⚠️  Você não digitou nada!")
        return
    
    resultados = buscar_aluno(termo)
    mostrar_alunos(resultados, f"RESULTADOS PARA: '{termo}'")
def atualizar_interface():
    """
    FUNÇÃO: atualizar_interface()
    OBJETIVO: Interface para atualizar dados de um aluno
    """
    print("\n" + "=" * 50)
    print("        ATUALIZAR DADOS DO ALUNO")
    print("=" * 50)
    
    # Primeiro mostra todos os alunos
    alunos = listar_alunos()
    if not alunos:
        print("\n  ⚠️  Nenhum aluno cadastrado para atualizar!")
        return
    
    print("\n  Alunos cadastrados:")
    for aluno in alunos:
        print(f"    ID {aluno[0]} → {aluno[1]} (Mat: {aluno[2]})")
    
    # Pega o ID do aluno
    try:
        id_aluno = int(input("\n  Digite o ID do aluno que deseja atualizar: "))
    except ValueError:
        print("\n  ❌ ERRO: ID deve ser um número!")
        return
    
    # Busca os dados atuais do aluno
    conexao = conectar()
    cursor = conexao.cursor()
    cursor.execute("SELECT nome, curso, nota1, nota2 FROM alunos WHERE id = ?", (id_aluno,))
    dados_atuais = cursor.fetchone()
    conexao.close()
    
    if not dados_atuais:
        print("\n  ❌ ERRO: Aluno não encontrado!")
        return
    
    # Mostra os dados atuais
    print(f"\n  Dados atuais do aluno:")
    print(f"    Nome: {dados_atuais[0]}")
    print(f"    Curso: {dados_atuais[1]}")
    print(f"    Notas: {dados_atuais[2]} e {dados_atuais[3]}")
    
    print("\n  (Deixe em branco para manter o valor atual)")
    
    # Coleta os novos dados
    novo_nome = input(f"  Novo nome [{dados_atuais[0]}]: ").strip()
    novo_curso = input(f"  Novo curso [{dados_atuais[1]}]: ").strip()
    
    try:
        nova_nota1_str = input(f"  Nova nota 1 [{dados_atuais[2]}]: ").strip()
        nova_nota2_str = input(f"  Nova nota 2 [{dados_atuais[3]}]: ").strip()
        
        # Se deixou em branco, mantém o valor antigo
        nova_nota1 = float(nova_nota1_str) if nova_nota1_str else dados_atuais[2]
        nova_nota2 = float(nova_nota2_str) if nova_nota2_str else dados_atuais[3]
    except ValueError:
        print("\n  ❌ ERRO: Notas devem ser números!")
        return
    
    # Define os valores finais
    nome_final = novo_nome if novo_nome else dados_atuais[0]
    curso_final = novo_curso if novo_curso else dados_atuais[1]
    
    # Atualiza o aluno
    atualizar_aluno(id_aluno, nome_final, curso_final, nova_nota1, nova_nota2)
    print("\n  ✅ DADOS ATUALIZADOS COM SUCESSO!")
def excluir_interface():
    """
    FUNÇÃO: excluir_interface()
    OBJETIVO: Interface para excluir um aluno
    """
    print("\n" + "=" * 50)
    print("        EXCLUIR ALUNO")
    print("=" * 50)
    
    # Primeiro mostra todos os alunos
    alunos = listar_alunos()
    if not alunos:
        print("\n  ⚠️  Nenhum aluno cadastrado para excluir!")
        return
    
    print("\n  Alunos cadastrados:")
    for aluno in alunos:
        print(f"    ID {aluno[0]} → {aluno[1]} (Mat: {aluno[2]})")
    
    # Pega o ID do aluno
    try:
        id_aluno = int(input("\n  Digite o ID do aluno que deseja EXCLUIR: "))
    except ValueError:
        print("\n  ❌ ERRO: ID deve ser um número!")
        return
    
    # Busca o nome do aluno para confirmar
    conexao = conectar()
    cursor = conexao.cursor()
    cursor.execute("SELECT nome FROM alunos WHERE id = ?", (id_aluno,))
    aluno = cursor.fetchone()
    conexao.close()
    
    if not aluno:
        print("\n  ❌ ERRO: Aluno não encontrado!")
        return
    
    # Confirmação (para evitar exclusão acidental)
    print(f"\n  ⚠️  ATENÇÃO: Você está prestes a excluir o aluno: {aluno[0]}")
    confirmacao = input("  Digite 'SIM' para confirmar a exclusão: ").strip().upper()
    
    if confirmacao == "SIM":
        excluir_aluno(id_aluno)
        print("\n  ✅ ALUNO EXCLUÍDO COM SUCESSO!")
    else:
        print("\n  ❌ EXCLUSÃO CANCELADA!")
# ================================================================
# PARTE 4: INTERFACE COM O USUÁRIO (TUDO QUE APARECE NA TELA)
# ================================================================
def main():
    """
    FUNÇÃO: main()
    OBJETIVO: Função principal que controla todo o fluxo do programa
    OBSERVAÇÃO: Diferente da linguagem C, aqui main() é apenas uma convenção
    """
    
    # Passo 1: Criar a tabela no banco de dados (se não existir)
    criar_tabela()
    
    # Passo 2: Exibir mensagem de boas-vindas
    print("\n" + "=" * 50)
    print("   BEM-VINDO AO SISTEMA DE REGISTRO DE ALUNOS")
    print("=" * 50)
    print("   Sistema iniciado com sucesso!")
    print("   Banco de dados: escola.db")
    
    # Passo 3: Loop principal (roda até o usuário escolher sair)
    while True:
        exibir_menu()
        
        opcao = input("  Digite sua opção: ").strip()
        
        # CRUD - CREATE
        if opcao == "1":
            cadastrar_interface()
        
        # CRUD - READ (listar todos)
        elif opcao == "2":
            listar_interface()
        
        # CRUD - READ (buscar específico)
        elif opcao == "3":
            buscar_interface()
        
        # CRUD - UPDATE
        elif opcao == "4":
            atualizar_interface()
        
        # CRUD - DELETE
        elif opcao == "5":
            excluir_interface()
        
        # SAIR DO SISTEMA
        elif opcao == "0":
            print("\n" + "=" * 50)
            print("   OBRIGADO POR USAR O SISTEMA!")
            print("   Até logo! 👋")
            print("=" * 50)
            break  # Sai do loop while
        
        # OPÇÃO INVÁLIDA
        else:
            print("\n  ❌ OPÇÃO INVÁLIDA!")
            print("  Digite 1, 2, 3, 4, 5 ou 0")
        
        # Pausa antes de limpar a tela (para o usuário ler as mensagens)
        input("\n  Pressione ENTER para continuar...")
        
        # Limpa a tela (funciona no Windows, Linux e Mac)
        import os
        os.system("cls" if os.name == "nt" else "clear")
# ================================================================
# PONTO DE ENTRADA DO PROGRAMA
# ================================================================
# Esta linha verifica se o arquivo está sendo executado diretamente
# (e não importado como módulo por outro programa)
if __name__ == "__main__":
    main()  # Aqui o programa COMEÇA A RODAR
