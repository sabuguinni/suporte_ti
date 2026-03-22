# Projeto: Sistema de Suporte Automático ao Aluno — Transpersonal International
**Documento de referência para desenvolvimento contínuo no Claude Project**
*Versão 1.0 — Março 2026*

---

## 1. Contexto e ponto de partida

### Quem somos
A **Transpersonal International** é uma escola de formação em terapias integrativas, hipnoterapia e desenvolvimento humano. O fundador e principal estratega é **Hugo Martins**, que assina todos os materiais de comunicação e lidera o desenvolvimento tecnológico da escola.

### Infraestrutura existente
- **Plataforma LMS + CRM** — construída com o Manus, que usa Claude como motor de raciocínio nativo.
- **Sistema de correção automática de trabalhos** — já operacional. Os alunos enviam exercícios escritos, os manuais dos cursos estão carregados, e a IA responde com correções. Funcionou inicialmente com aprovação obrigatória dos formadores; após 5 respostas consecutivas sem edição, passou a responder de forma autónoma. Eficácia e precisão elevadas.
- **Relevance AI** — plataforma de agentes de IA já contratada e em uso. Tem knowledge bases carregadas. É aqui que o novo sistema de suporte será construído.

### O que se pretende construir
Um sistema de **suporte automático ao aluno** que:
1. Recebe emails de suporte
2. Classifica o tipo de dúvida
3. Encaminha para o agente especializado correto
4. Responde automaticamente com base no conhecimento carregado
5. Escala para humano quando necessário
6. Aprende progressivamente, com o mesmo modelo de aprovação usado na correção de trabalhos

---

## 2. Arquitetura do sistema

### Fluxo principal

```
Email de suporte recebido
        ↓
  [Agente Triagem]
  Classifica a dúvida em categoria
        ↓
   ┌─────────────┬──────────────┬─────────────┐
   ↓             ↓              ↓             ↓
[Agente        [Agente       [Agente       [Agente
Dúvidas        Plataforma/   Administrativo Emocional/
de Curso]      Acesso]       /Pagamentos]  Processo]
   ↓             ↓              ↓             ↓
Resposta     Resposta       Escalamento   Escalamento
automática   automática     humano        humano
(RAG manuais) (RAG FAQ)     (maioria)     (sempre)
```

### Integração técnica
- **Entrada de email**: via Make (ex-Integromat) ou Zapier — email entra → webhook → Relevance AI
- **Saída**: resposta enviada automaticamente pelo mesmo canal
- **Aprovação humana**: ativada por defeito no início; desativada por categoria após threshold de consistência

---

## 3. Agentes — definição e responsabilidades

### Agente 0 — Triagem
- **Função**: Ler o email, identificar a categoria, encaminhar para o agente correto
- **Output**: JSON com `{categoria, confiança, resumo_da_dúvida, agente_destino}`
- **Regra**: Se confiança < 70%, escala para humano em vez de encaminhar
- **Conhecimento**: Exemplos de cada categoria (criados a partir das conversas de WhatsApp)

### Agente 1 — Dúvidas de Curso
- **Função**: Responder a dúvidas sobre conteúdo, módulos, conceitos, exercícios
- **Knowledge base**: Manuais dos cursos + pares Q&A extraídos de conversas de WhatsApp (categoria: conteúdo)
- **Tom**: Próximo, pedagógico, alinhado com a voz da Transpersonal International
- **Escalamento**: Se a dúvida exigir acompanhamento personalizado de um formador

### Agente 2 — Plataforma e Acesso
- **Função**: Resolver problemas técnicos, acesso a módulos, downloads, passwords, navegação
- **Knowledge base**: FAQ técnico + conversas de WhatsApp (categoria: acesso/técnico)
- **Tom**: Claro, passo a passo, sem jargão
- **Escalamento**: Problemas que não estão no FAQ ou que persistem após as instruções

### Agente 3 — Administrativo e Pagamentos
- **Função**: Questões de faturação, prazos, certificados, inscrições, cancelamentos
- **Knowledge base**: Políticas internas, FAQs administrativos
- **Tom**: Formal mas acolhedor
- **Escalamento**: Quase sempre — responde apenas ao que é inequívoco (ex: "quando recebo o certificado?")

### Agente 4 — Processo Emocional
- **Função**: Identificar e acolher mensagens em que o aluno expressa dificuldade emocional, bloqueio no processo terapêutico ou crise
- **Knowledge base**: Não tem knowledge base de respostas — tem apenas guidelines de como acolher e encaminhar
- **Tom**: Caloroso, sem diagnóstico, sem conselho terapêutico
- **Escalamento**: Sempre — o agente acolhe e encaminha para formador humano

---

## 4. Modelo de aprendizagem progressiva

Replicar o modelo já validado na correção de trabalhos:

| Fase | Condição | Comportamento |
|------|----------|---------------|
| Fase 1 | Início | Toda a resposta exige aprovação do formador antes de ser enviada |
| Fase 2 | 5 respostas sem edição (por categoria) | Passa a responder autonomamente nessa categoria |
| Fase 3 | Monitorização contínua | Qualquer edição humana reverte para Fase 1 nessa categoria |
| Fase 4 | Revisão mensal | Análise de qualidade, ajuste de knowledge bases |

---

## 5. Knowledge bases — fontes e estado atual

### Fontes disponíveis

| Fonte | Estado | Próximo passo |
|-------|--------|---------------|
| Manuais dos cursos | Já carregados na Relevance | Confirmar cobertura por curso |
| Conversas de WhatsApp (grupos de alunos) | Exportadas em .txt | Anonimizar + extrair Q&A + categorizar |
| Emails de suporte anteriores | A verificar | Exportar se disponível |
| FAQ técnico da plataforma | A criar | Redigir com base nos problemas mais comuns |
| Políticas administrativas | A criar | Compilar documento interno |

### Processo de tratamento das conversas de WhatsApp

1. Hugo exporta cada grupo: *Definições do grupo → Exportar conversa → Sem média*
2. Carrega os ficheiros `.txt` neste projeto
3. Claude processa cada ficheiro:
   - Remove todos os nomes próprios (substitui por "Aluno", "Formador", "Aluno A", "Aluno B"...)
   - Remove mensagens irrelevantes (saudações, emojis isolados, off-topic)
   - Extrai pares pergunta/resposta relevantes
   - Formata em estrutura Q&A para RAG:
     ```
     Dúvida: [texto da pergunta, anonimizado]
     Contexto: [se relevante — módulo, fase do curso]
     Resposta: [resposta dada, anonimizada]
     Categoria: [conteúdo / acesso / administrativo / emocional]
     ```
4. Entrega ficheiro `.md` limpo por grupo, pronto a carregar na Relevance

---

## 6. Prompts base dos agentes (a desenvolver)

Esta secção será preenchida ao longo do projeto. Cada agente terá:
- System prompt completo
- Exemplos few-shot (3 a 5 por agente)
- Instruções de escalamento
- Tom e linguagem alinhados com o brandbook da Transpersonal International

**Estado atual**: A desenvolver. Será feito neste projeto após tratamento das conversas de WhatsApp.

---

## 7. Integração técnica — passos de implementação

### Fase 1 — Preparação (a fazer agora)
- [ ] Exportar todas as conversas de WhatsApp relevantes
- [ ] Tratar ficheiros com Claude (anonimizar + Q&A)
- [ ] Criar FAQ técnico da plataforma
- [ ] Compilar políticas administrativas

### Fase 2 — Construção na Relevance
- [ ] Criar knowledge base por categoria
- [ ] Criar Agente Triagem com prompt de classificação
- [ ] Criar Agentes 1 a 4 com prompts e knowledge bases
- [ ] Configurar fluxo de aprovação humana

### Fase 3 — Integração de email
- [ ] Configurar Make/Zapier: email entra → webhook Relevance
- [ ] Configurar resposta automática via mesmo canal
- [ ] Testar com emails reais (modo aprovação)

### Fase 4 — Activação progressiva
- [ ] Monitorizar aprovações por categoria
- [ ] Activar autonomia categoria a categoria após threshold
- [ ] Revisão mensal de qualidade

---

## 8. Regras de trabalho neste projeto

- Toda a comunicação é em **português de Portugal** (novo acordo ortográfico)
- Sem gerúndios, sem linguagem brasileira
- Tratar Hugo na segunda pessoa do singular
- Respostas com linguagem falada e humana
- Edições cirúrgicas — não reescrever o que não foi pedido
- Quando forem dados exemplos de prompts ou estruturas, apresentar de forma explicativa e passo a passo
- A voz dos agentes deve estar alinhada com o brandbook da Transpersonal International (carregar brandbook neste projeto quando disponível)

---

## 9. Documentos a carregar neste projeto

Para maximizar a eficácia do trabalho aqui desenvolvido, carregar progressivamente:

- [ ] Brandbook completo da Transpersonal International
- [ ] Manual de avatar de cliente (8 arquétipos psicográficos)
- [ ] Manuais dos cursos (ou sumários)
- [ ] Conversas de WhatsApp exportadas (.txt)
- [ ] Emails de suporte anteriores (se disponíveis)
- [ ] Qualquer prompt ou instrução já criada para os agentes Relevance existentes

---

*Este documento é vivo — será actualizado ao longo do desenvolvimento do projeto.*
