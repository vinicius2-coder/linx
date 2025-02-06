---
title: "Utilizando `screen` no Terminal"
date: 2020-09-15T11:30:03+00:00
author: "Alessandro César Rosão"
categories: ["Linux", "Terminal", "Produtividade"]
tags: ["screen", "terminal", "automação", "servidor"]
---

O `screen` é uma ferramenta que permite criar sessões de terminal que continuam sendo executadas em segundo plano, mesmo que você se desconecte ou feche o terminal. Ele é especialmente útil em servidores remotos ou ao executar tarefas de longa duração.  

## Instalação  

No **Ubuntu/Debian** e derivados, você pode instalar o `screen` com:  

```bash
sudo apt install screen
```

No **Arch Linux**:  

```bash
sudo pacman -S screen
```

## Criando e usando sessões `screen`  

### 1. Criar uma nova sessão  

Para iniciar uma nova sessão de `screen`:  

```bash
screen
```

Isso abrirá um novo terminal dentro da sessão. Para sair temporariamente da sessão (desanexar), veja o próximo passo.  

### 2. Desanexar uma sessão  

Para "desanexar" (desconectar) uma sessão de `screen` sem encerrá-la, pressione:  

```
Ctrl + A, depois D
```

Isso o retornará ao terminal normal, enquanto a sessão do `screen` continua rodando em segundo plano.  

### 3. Listar sessões ativas  

Para ver todas as sessões de `screen` ativas:  

```bash
screen -ls
```

### 4. Reanexar uma sessão  

Para voltar a uma sessão desanexada:  

```bash
screen -r <ID_da_sessão>
```

Por exemplo, se a sessão tiver o ID `12345.pts-0.servidor`:  

```bash
screen -r 12345
```

### 5. Criar uma sessão nomeada  

Para facilitar a identificação, você pode nomear suas sessões:  

```bash
screen -S minha_sessao
```

E para reconectar:  

```bash
screen -r minha_sessao
```

### 6. Fechar uma sessão  

Para fechar uma sessão, basta sair do terminal dentro dela com:  

```bash
exit
```

Ou pressionando `Ctrl + D`.  

### 7. Excluir uma sessão específica  

```bash
screen -S nome_da_sessao -X quit
```

## Comandos úteis do `screen`  

Dentro de uma sessão `screen`, utilize **Ctrl + A** seguido de outro comando:  

| Comando               | Descrição                                      |
|-----------------------|------------------------------------------------|
| **Ctrl + A, D**       | Desanexa a sessão atual                       |
| **Ctrl + A, Shift + A** | Renomeia a janela atual                     |
| **Ctrl + A, Shift + "** | Volta ao hub de janelas                     |
| **Ctrl + A, K**       | Remove a janela da listagem                   |
| **Ctrl + A, :number** | Altera a posição dela na listagem             |
| **Ctrl + A, C**       | Cria uma nova janela dentro do hub            |
| **Ctrl + A, N**       | Alterna para a próxima janela                 |
| **Ctrl + A, P**       | Alterna para a janela anterior                |
| **Ctrl + A, K**       | Fecha a janela atual                          |
| **Ctrl + A, W**       | Mostra uma lista de janelas                   |
| **Ctrl + A, [**       | Ativa o modo de cópia para copiar texto       |
| **Ctrl + A, ?**       | Exibe a ajuda do `screen`                     |

## Executando comandos no `screen`  

Você pode iniciar um comando diretamente dentro de uma sessão `screen` sem entrar nela. Por exemplo, para rodar um script chamado `script.sh`:  

```bash
screen -S minha_sessao -dm bash -c './script.sh'
```

- `-S`: Nomeia a sessão.  
- `-dm`: Inicia a sessão em modo desanexado.  
- `-c`: Executa o comando especificado.  

## Casos de uso práticos  

1. **Execução de tarefas de longa duração**  
   - Se você estiver rodando um script importante, pode iniciá-lo em um `screen` para evitar a interrupção caso a conexão com o servidor caia.  

2. **Gerenciamento de múltiplas sessões**  
   - Permite abrir várias sessões e alternar entre elas, útil para monitoramento de logs e execução de comandos simultaneamente.  

3. **Compartilhamento de sessões**  
   - Para permitir que outros usuários do sistema acessem uma sessão ativa:  

   ```bash
   screen -x
   ```

## Conclusão  

O `screen` é uma ferramenta essencial para administração de servidores e gerenciamento de sessões remotas. Com sua flexibilidade e comandos intuitivos, ele facilita a execução de tarefas de longa duração e o multitasking no terminal.  