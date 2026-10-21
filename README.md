# 💡 Apague as Luzes

<p align="center">
  <strong>Um desafio de lógica, estratégia e raciocínio.</strong>
</p>

<p align="center">
  Um jogo interativo em que cada movimento altera as luzes vizinhas. O objetivo é apagar todas as luzes utilizando estratégia e o menor número possível de movimentos.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Vue.js-3-42b883?logo=vuedotjs&logoColor=white" alt="Vue 3" />
  <img src="https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white" alt="TypeScript" />
  <img src="https://img.shields.io/badge/Vite-646CFF?logo=vite&logoColor=white" alt="Vite" />
  <img src="https://img.shields.io/badge/Status-Concluído-brightgreen" alt="Status do projeto" />
</p>

---

## 🎮 Sobre o projeto

**Apague as Luzes** é um jogo de lógica desenvolvido para exercitar o raciocínio e o planejamento de movimentos por meio de uma interface interativa.

Ao selecionar uma luz, seu estado é invertido, assim como o das luzes adjacentes na horizontal e na vertical. O desafio termina quando todas as luzes estão apagadas.

O projeto foi desenvolvido com foco em uma experiência intuitiva, identidade visual consistente, adaptação a diferentes tamanhos de tela e acompanhamento do desempenho durante a partida.

## ✨ Funcionalidades

- **Tabuleiro interativo:** altere o estado das luzes com cliques.
- **Níveis de dificuldade:** escolha entre Fácil, Médio e Difícil.
- **Estratégia e raciocínio:** resolva o desafio utilizando a menor quantidade possível de movimentos.
- **Sistema de dicas:** receba auxílio para encontrar o próximo movimento.
- **Indicadores da partida:** acompanhe movimentos, tempo decorrido e progresso.
- **Reinício de partida:** recomece o desafio mantendo a configuração atual.
- **Personalização visual:** alterne entre seis temas de cores.
- **Modal de conclusão:** visualize as estatísticas da partida ao vencer.
- **Interface responsiva:** layout adaptado para computadores, tablets e smartphones.

## 🛠️ Tecnologias utilizadas

| Tecnologia | Aplicação |
|---|---|
| Vue 3 | Construção da interface e gerenciamento reativo do estado |
| TypeScript | Tipagem estática e maior segurança no desenvolvimento |
| Vite | Ambiente de desenvolvimento e geração da versão de produção |
| HTML5 | Estrutura da aplicação |
| CSS3 | Estilização, temas, animações e responsividade |
| Git e GitHub | Versionamento e hospedagem do código-fonte |

## 🎨 Temas visuais

O jogo oferece seis opções de aparência:

- 🟢 Verde
- 🟣 Roxo Arcade
- 🟠 Laranja Solar
- 🧊 Azul Gelo
- 🌐 Grafite
- 🏜️ Areia

A troca de tema modifica a apresentação visual sem reiniciar a partida.

## 🚀 Como executar o projeto

### Pré-requisitos

- [Node.js](https://nodejs.org/) instalado.
- npm, incluído na instalação do Node.js.
- Git para clonar o repositório.

### 1. Clone o repositório

```bash
git clone https://github.com/RicardoSpicka/apague-as-luzes.git
```

### 2. Acesse a pasta

```bash
cd apague-as-luzes
```

### 3. Instale as dependências

```bash
npm install
```

### 4. Execute em ambiente de desenvolvimento

```bash
npm run dev
```

O Vite exibirá no terminal o endereço local para abrir a aplicação no navegador.

### 5. Gere a versão de produção

```bash
npm run build
```

Os arquivos gerados serão disponibilizados no diretório `dist/`.

Para testar a versão de produção localmente:

```bash
npm run preview
```

## 🧠 Regras do jogo

1. Cada luz pode estar acesa ou apagada.
2. Ao selecionar uma luz, seu estado é invertido.
3. As luzes imediatamente adjacentes na horizontal e na vertical também têm seus estados invertidos.
4. Luzes diagonais não são afetadas.
5. O objetivo é apagar todas as luzes do tabuleiro.
6. Ao concluir o desafio, o jogo apresenta um resumo da partida.

**Dica:** observe como cada movimento afeta