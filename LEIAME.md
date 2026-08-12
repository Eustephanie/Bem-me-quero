# Bem Me Quero by Marcia — site

Site do salão, feito em HTML/CSS/JavaScript puro. Não precisa instalar nada:
é só abrir o `index.html` no navegador (duplo clique) que ele funciona.

---

## O que ainda falta ajustar antes de publicar

Tudo isso está no arquivo **`js/config.js`** — abra com o Bloco de Notas
(ou VS Code) e edite o que estiver marcado.

### 1. O número do WhatsApp ✅ já configurado

```js
whatsapp: "5511983743557",
whatsappVisivel: "(11) 98374-3557",
```

O formato é **55 + DDD + número**, só dígitos, sem espaços nem traços.
Se um dia mudar o número, é aqui.

### 2. O endereço completo

```js
endereco: "Rua Saverio Quadrio, 250", // <-- COMPLETE com bairro e cidade
```

Se quiser, cole também o link do Google Maps em `linkMapa`.

### 3. Os preços e a duração de cada serviço

Na lista `servicos`. Os valores que estão lá são **exemplos** — confira todos.
A `duracao` é em minutos e é o que faz o site calcular os horários livres,
então precisa estar realista.

---

## Como funciona o agendamento

A cliente escolhe **um ou mais serviços → dia → horário → nome e telefone**.
Ao clicar em "Enviar pelo WhatsApp", abre o WhatsApp da Marcia com a mensagem
já pronta:

```
Olá, Marcia! Quero agendar um horário pelo site 💜

*Nome:* Ana Paula
*Telefone:* (11) 98765-4321
*Serviços:*
• Alongamento em gel — R$ 150 (2h30)
• Design de sobrancelha — R$ 40 (30 min)
*Valor total:* R$ 190
*Duração total:* 3 horas
*Data:* Sexta-feira, 14 de agosto (14/08/2026)
*Horário:* 09:00 às 12:00

Aguardo a confirmação. Obrigada!
```

**A confirmação é sempre manual, pelo WhatsApp.** O site mostra os horários
possíveis, mas quem confirma é a Marcia.

### Vários serviços de uma vez

A cliente pode marcar quantos serviços quiser (é uma caixinha de seleção em
cada um). O site então:

- **soma as durações** e só oferece horários em que tudo cabe.
  Ex.: alongamento (2h30) + sobrancelha (30 min) = 3 horas, então às 17:00
  o horário some, porque não daria tempo antes de fechar;
- **soma os valores** e mostra o total;
- mostra o **horário de término** ("das 09:00 às 12:00");
- se algum serviço for "A combinar", o total aparece como
  "R$ 190 + a combinar".

### Configurando a agenda (`js/config.js` → `agenda`)

| O que | Onde | Para que serve |
|---|---|---|
| Dias e horas de trabalho | `expediente` | `null` = fechado naquele dia |
| Almoço | `pausa` dentro de cada dia | Nenhum serviço atravessa o almoço |
| De quanto em quanto tempo | `intervaloMinutos` | 30 = horários de meia em meia hora |
| Antecedência mínima | `antecedenciaMinimaHoras` | 12 = ninguém marca para daqui a 2 horas |
| Até quando dá para marcar | `diasNoFuturo` | `null` = sem limite (qualquer mês e ano) |
| Anos na listinha | `anosNoSeletor` | 10 = de hoje até daqui a 10 anos |
| Férias / feriados | `datasBloqueadas` | Fecha o dia inteiro |
| Horários já ocupados | `horariosOcupados` | Some do site aquele horário específico |

### O calendário

O calendário é livre: tem **listinha de mês e de ano**, então a cliente pode
ir para qualquer data — dezembro de 2030, se quiser. As setas ‹ › andam mês a
mês. Datas passadas aparecem, mas apagadas e sem clique (não dá para marcar
ontem). Domingos e segundas ficam apagados porque o salão fecha.

Exemplo de dia bloqueado e horários ocupados:

```js
datasBloqueadas: ["2026-12-25", "2027-01-01"],

horariosOcupados: {
  "2026-08-14": ["09:00", "13:00"]
}
```

---

## Integração com o Google Agenda

O site já sabe conversar com o Google Agenda. Quando você liga essa opção,
**tudo que estiver marcado na agenda da Marcia some automaticamente do site** —
sem precisar anotar nada à mão.

### O que a integração faz e o que NÃO faz

| Faz | Não faz |
|---|---|
| **Lê** a agenda e esconde os horários ocupados | **Não escreve** na agenda sozinho |
| Some com o dia inteiro se tiver evento de dia inteiro (férias, médico) | Não cria o compromisso quando a cliente envia o pedido |
| Entende compromissos que se repetem toda semana | |
| Ignora eventos marcados como "Disponível" no Google | |

Então o fluxo do dia a dia fica assim:

1. A cliente escolhe o horário no site e envia pelo WhatsApp
2. A Marcia confirma pelo WhatsApp
3. **A Marcia cria o compromisso no Google Agenda** (pelo celular, leva 10 s)
4. Pronto: aquele horário some do site para todas as outras clientes

O passo 3 é manual porque um site sem servidor não pode ter permissão de
escrever na agenda de alguém — a senha do Google teria que ficar no código,
o que seria inseguro. Se um dia quiser que o site marque sozinho, aí sim
precisa de um sistema com banco de dados; me chame que eu monto.

### Como ligar (uma vez só, ~15 minutos)

**1. Crie um calendário separado só para o salão**

No Google Agenda (computador): *Outras agendas* → **+** → *Criar nova agenda*.
Chame de "Bem Me Quero — Atendimentos". Use um calendário novo, **não o
pessoal dela**.

**2. Deixe esse calendário público, só como livre/ocupado**

Configurações da agenda → *Permissões de acesso* → marque
**"Disponibilizar publicamente"** e, na listinha ao lado, escolha
**"Ver apenas informações de livre/ocupado (ocultar detalhes)"**.

> ⚠️ **Privacidade:** isso é obrigatório para o site conseguir ler a agenda.
> Escolhendo "livre/ocupado", ninguém vê o nome das clientes nem o que está
> marcado — só que aquele horário está ocupado. Por isso é importante usar um
> calendário separado, e não a agenda pessoal dela.

**3. Copie o ID do calendário**

Na mesma tela, em *Integrar agenda*, copie o **ID da agenda**. É algo como
`c_a1b2c3...@group.calendar.google.com`.

**4. Crie a chave de API**

- Acesse <https://console.cloud.google.com/>
- Crie um projeto (nome livre, ex.: "Site Bem Me Quero")
- Menu *APIs e serviços* → *Biblioteca* → procure **Google Calendar API** →
  **Ativar**
- *APIs e serviços* → *Credenciais* → **Criar credenciais** → **Chave de API**
- Copie a chave (`AIzaSy...`)
- Clique em *Restringir chave*: em **Restrições de aplicativo** escolha
  "Sites" e adicione o endereço do site (`bemmequero.com.br/*`);
  em **Restrições de API** deixe só a Google Calendar API

> A chave fica visível no código do site — é assim mesmo, ela é feita para
> isso. As restrições acima é que impedem que alguém use a chave em outro site.

**5. Cole tudo no `js/config.js`**

```js
googleAgenda: {
  ativo: true,
  calendarId: "c_a1b2c3...@group.calendar.google.com",
  apiKey: "AIzaSy...",
  fusoHorario: "America/Sao_Paulo"
}
```

Publique o site de novo e pronto. Enquanto ele consulta a agenda aparece
"Conferindo a agenda da Marcia…" embaixo do calendário.

### Se der errado

O site **nunca quebra** por causa disso. Se a chave estiver errada, a agenda
não estiver pública ou faltar internet, ele mostra um aviso discreto
("Não conseguimos conferir a agenda agora") e continua funcionando com os
horários configurados no `expediente` — exatamente como antes.

Para desligar a integração, é só voltar `ativo: false`.

---

## E enquanto o Google Agenda não estiver ligado?

Com a integração **desligada**, o site não sabe o que já foi marcado: duas
clientes podem pedir as 14:00 do mesmo dia. Como quem confirma é sempre a
Marcia pelo WhatsApp, não vira marcação errada — mas dá o trabalho de
remarcar uma delas.

Nesse caso, anote os horários confirmados à mão e publique de novo:

```js
horariosOcupados: {
  "2026-08-14": ["09:00", "13:00"],
  "2026-08-15": ["10:00"]
}
```

Se um dia inteiro lotar, use `datasBloqueadas`, que é mais rápido.

**É justamente esse trabalho manual que a integração com o Google Agenda
elimina** — por isso vale os 15 minutos de configuração.

---

## As fotos da galeria

A galeria já está com **8 fotos reais**, baixadas do Instagram
[@bemmequerobymarcia](https://instagram.com/bemmequerobymarcia) — são
`imagens/unha-01.jpg` até `unha-09.jpg`. A `unha-08.jpg` é a que aparece na
seção "Sobre" (tem o ambiente do salão ao fundo).

> As fotos vieram na resolução que o Instagram entrega (480 × 640). Se você
> tiver os **originais** no celular, vale trocar: ficam bem mais nítidas.
> Deixei de fora um vídeo e uma foto repostada de outra pessoa.

### Para trocar ou acrescentar fotos

1. Salve as fotos na pasta **`imagens`** (formato `.jpg` ou `.webp`,
   de preferência em **retrato** e com menos de 500 KB cada).
2. Em `js/config.js`, na lista `galeria`, troque o caminho:

```js
galeria: [
  { src: "imagens/unha-01.jpg", alt: "Alongamento em gel nude", categoria: "gel" },
  { src: "imagens/unha-02.jpg", alt: "Nail art com pedrarias",  categoria: "nailart" }
]
```

- `alt` = descrição da foto (aparece ao passar o mouse e ajuda no Google).
- `categoria` = `gel`, `nailart`, `francesinha`, `pes` ou `outros`
  (são os botões de filtro). Os nomes dos filtros ficam logo abaixo, em
  `categoriasGaleria`.
- Pode ter quantas fotos quiser — é só acrescentar linhas.

**A foto da seção "Sobre"** é a `imagens/unha-08.jpg`. Para trocar por uma foto
da Marcia ou do salão, ajuste o caminho em `index-fonte.html`
(procure por `imagens/unha-08.jpg`) e gere o `index.html` de novo.

**A logo** é a `imagens/logo.jpg`, e aparece no cabeçalho e no rodapé. Para
trocar, basta substituir esse arquivo mantendo o mesmo nome e gerar de novo.

---

## Depoimentos de clientes

A seção só aparece se você preencher. Em `js/config.js`:

```js
depoimentos: [
  { nome: "Ana P.", texto: "Minhas unhas nunca duraram tanto!" },
  { nome: "Carla M.", texto: "Atendimento impecável, saí amando." }
  ]
  ```

Use **somente depoimentos reais**, de clientes que autorizaram.

---

## Como colocar o site no ar (de graça, no GitHub Pages)

O site fica hospedado no **GitHub Pages**, no endereço
**<https://bemmequero.com.br>**. É gratuito e não tem limite de visitas.

### 1. Criar o repositório (uma vez só)

1. Crie uma conta em <https://github.com> (se ainda não tiver)
2. Clique em **New repository**
3. Nome: `bem-me-quero` · Visibilidade: **Public**
   (o Pages gratuito só funciona em repositório público — só o código do site
   fica visível, e ele não guarda nenhum dado de cliente)
4. **Não** marque "Add a README" — a pasta já tem tudo
5. Clique em **Create repository**

### 2. Enviar a pasta

O GitHub mostra os comandos na tela do repositório novo. Na pasta do site:

```bash
git remote add origin https://github.com/SEU-USUARIO/bem-me-quero.git
git push -u origin main
```

Trocando `SEU-USUARIO` pelo seu nome de usuário do GitHub.

### 3. Ligar o Pages

No repositório: **Settings** → **Pages** → em *Source* escolha
**Deploy from a branch**, branch **main**, pasta **/ (root)** → **Save**.

Em um ou dois minutos o site está no ar.

### 4. Apontar o domínio bemmequero.com.br

O arquivo `CNAME` da pasta já avisa o GitHub qual é o domínio. Falta só apontar
o domínio para o GitHub, no **Registro.br** (onde o `.com.br` foi registrado):

Entre em <https://registro.br> → seu domínio → **Editar zona / DNS** e cadastre:

| Tipo | Nome | Valor |
|---|---|---|
| A | (vazio, ou `@`) | `185.199.108.153` |
| A | (vazio, ou `@`) | `185.199.109.153` |
| A | (vazio, ou `@`) | `185.199.110.153` |
| A | (vazio, ou `@`) | `185.199.111.153` |
| CNAME | `www` | `SEU-USUARIO.github.io.` |

São os quatro endereços do GitHub — cadastre **todos**, é o que mantém o site
no ar se um deles cair.

Depois volte em **Settings** → **Pages** e marque **Enforce HTTPS** (o cadeado
do navegador). A opção só aparece depois que o domínio começa a funcionar.

> ⏱️ A mudança de DNS leva de alguns minutos até algumas horas para valer no
> mundo todo. É normal o site demorar um pouco para abrir no domínio novo.

### Os dois arquivos: qual é qual

Na pasta existem dois arquivos parecidos, e a diferença é o que faz o site
subir certo ou errado:

| Arquivo | O que é | Sobe? |
|---|---|---|
| **`index.html`** | O site inteiro dentro de um arquivo só: estilos, código e fotos | ✅ **É este que você sobe** |
| `index-fonte.html` | A receita, que só funciona com as pastas `css`, `js` e `imagens` ao lado | ❌ Nunca |

O `index.html` **funciona sozinho**: dá para subir só ele, ou abrir no
computador com dois cliques, sem internet e sem servidor. Suba junto o `CNAME`,
que é o arquivo que prende o domínio ao site.

> ⚠️ **Não teste o site mandando o arquivo por WhatsApp para o celular.** Ao
> tocar num `.html` recebido, o celular costuma abrir um visualizador de
> documentos, que não executa JavaScript e bloqueia as fotos embutidas: o site
> aparece quebrado mesmo estando perfeito. Para conferir no celular, publique
> e abra o endereço do site no navegador.

### Depois de mudar qualquer coisa, gere de novo

Sempre que editar preços, horários ou fotos no `js/config.js`, o `index.html`
precisa ser refeito, senão ele continua com o conteúdo antigo:

Clique com o botão direito em `ferramentas/gerar-arquivo-unico.ps1` →
*Executar com o PowerShell*.

> ⚠️ Nunca edite o `index.html` direto: ele é gerado automaticamente, e na
> próxima geração suas mudanças somem. Edite sempre o `js/config.js`.

### 5. Atualizar o site depois

Sempre que editar o `config.js` (preços, horários, fotos):

```bash
git add . && git commit -m "Atualiza horarios" && git push
```

O site no ar se atualiza sozinho em ~1 minuto. Se preferir não usar comandos,
dá para editar o arquivo direto pelo site do GitHub e clicar em *Commit changes*.

**Não esqueça:** coloque o link **bemmequero.com.br** na bio do Instagram — hoje
ela já convida a clicar no link.

---

## Cores

Estão todas no topo de `css/style.css`, no bloco `:root`:

```css
--papel:     #FBF8FC;   /* fundo, branco com um toque de lilás */
--papel-2:   #F5EFF7;   /* seções alternadas */
--escuro:    #1A1119;   /* faixa do contato e rodapé */
--tinta:     #1D141C;   /* cor dos textos */
--roxo:      #5B2A86;   /* roxo profundo: botões, links, detalhes */
--roxo-pale: #CDB5DE;   /* roxo claro para fundos escuros */
--linha:     #E6DCEA;   /* traços finos entre seções */
```

Mudou ali, mudou no site inteiro.

As fontes também estão logo abaixo: **Newsreader** (serifada, para os títulos)
e **DM Sans** (para os textos).

---

## Estrutura dos arquivos

```
Bem Me Quero/
├── index.html            ← É ESTE QUE VOCÊ SOBE (site inteiro num arquivo só)
├── index-fonte.html      a receita: só funciona com as pastas abaixo
├── css/style.css         cores e visual
├── js/config.js          ← É AQUI QUE VOCÊ EDITA (textos, preços, agenda, fotos)
├── js/script.js          lógica do agendamento e da galeria
├── js/google-agenda.js   conversa com o Google Agenda
├── imagens/              fotos, logo e ícones
├── CNAME                 prende o domínio bemmequero.com.br ao site
└── ferramentas/          gerador do arquivo único e servidor local
```

---

## Detalhes técnicos

- Funciona sem internet e sem servidor (basta abrir o `index.html`).
- Responsivo: testado em desktop e celular.
- Acessível: navegação por teclado, textos alternativos e leitores de tela.
- As fontes vêm do Google Fonts; sem internet o site usa fontes do sistema
  e continua funcionando.
