# Missão Indicação — UniCive Caruaru

Página do programa de indicação da UniCive. Uma pessoa indica outra, e as duas
aparecem na **mesma linha** da aba `INDICAÇÕES` da planilha de leads, com
`UNICIVE` na coluna BU.

Irmã da `indicaçõesCPPEM` e da `indicaçõesCOLEGIO`: mesma estrutura, mesma
lógica, mesmos ids. O que muda é o bloco `BU` no `script.js`, os tokens de cor
e o texto dos prêmios.

```
index.html     a página inteira (HTML + CSS)
script.js      validação, envio e o fluxo de "indicar outra pessoa"
public/        logo da UniCive e fotos de formandos
vercel.json    cache dos estáticos
```

---

## 1. Backend: nada a fazer

O Apps Script é **um só** para as três BUs, já está publicado e já reconhece
`INDICACAO_UNICIVE`. Esta pasta **não tem cópia dele de propósito**: duas
cópias do mesmo backend viram duas versões divergentes na primeira correção.

O arquivo vive em `../indicaçõesCPPEM/google-apps-script.js`, e o passo a passo
de publicação está no README de lá.

Testado em produção em 21/09/2026. Uma indicação enviada pela página caiu na
aba `INDICAÇÕES` com `BU = UNICIVE`, o telefone normalizado sem o `+` e a chave
Pix na coluna D.

---

## 2. Publicar a página

Site estático. Na Vercel, importar o repositório e publicar sem build.

Domínio no ar: **`indica.unicive.cppem.com.br`**, projeto Vercel
`indicacao-unicive`, conectado ao repositório (um `git push` publica sozinho).

> **Este domínio é o mais sensível dos três.** A regra `/unicive/i` da lista
> `CAMPANHAS` casa com o endereço inteiro. Sem uma regra de `DOMINIOS` que o
> reconheça primeiro, uma indicação que chegue sem `?aba=` é lida como
> campanha da UniCive e cai na aba `UNICIVE_Novo`, contada como lead de
> captura. Silenciosamente, na aba errada.
>
> Isso precisou de duas correções: em `resolverOrigem`, que deixou de procurar
> a campanha dentro da URL quando o domínio já foi reconhecido, e na própria
> lista `DOMINIOS`, atualizada de `indique.` para `indica.` quando o domínio
> real entrou no ar.
>
> **Se o domínio mudar de novo, a regra em `DOMINIOS` tem que mudar junto.**

---

## 3. O bloco BU

É o único ponto obrigatório ao replicar. Tudo que varia entre as três páginas
mora aqui:

```js
const BU = {
  chave: "INDICACAO_UNICIVE",   // decide a coluna BU no backend
  nome: "UNICIVE",              // vai junto nos eventos de dataLayer
  whatsapp: "5581992640766",
  whatsappMsg: "...",
  pedirChavePix: true,          // aqui o prêmio é pago em dinheiro
  selo: { ate: 60, prefixo: "", sufixo: "%" }
};
```

| | UniCive | CPPEM | Colégio |
|---|---|---|---|
| `chave` | `INDICACAO_UNICIVE` | `INDICACAO_CPPEM` | `INDICACAO_COLEGIO` |
| WhatsApp | `5581992640766` | `5581973105354` | `5581997076388` |
| `pedirChavePix` | `true` | `true` | `false` |
| `selo` | 60% | 10% | R$ 100 |

O selo mostra os **60% da matrícula**, que é a oferta de entrada. Os 73% da
mensalidade ficam na lista de preços ao lado, porque um anel só mostra um
número e o que puxa a conversa é o "R$ 99,90 para entrar".

---

## 4. Identidade visual

Esta é a **única das três com fundo claro**, e isso não é uma troca de cores: o
CSS inteiro foi invertido. Textos, bordas, sombras e os degradês de todos os
cartões mudam de lógica quando o fundo deixa de ser preto.

| | UniCive | CPPEM | Colégio |
|---|---|---|---|
| fundo | `#F2F4EC` (claro) | `#0a0a0b` | `#0D1B3E` |
| marca | `#006652` + `#FFC571` | `#af9256` | `#C9A227` |
| display | Bebas Neue | Oxanium | Cinzel |
| texto | Montserrat | Inter | DM Sans |

**O verde e o âmbar foram amostrados do PNG do logo**, não copiados do CSS da
`captura-unicive`. Lá o verde é `#4f7129`, um oliva que não é a cor da marca.

Três coisas que o fundo claro obrigou a mudar:

- **A faísca dos níveis** usava `mix-blend-mode: screen`, que clareia. Sobre um
  cartão branco não sobra nada para ver. Virou um brilho branco comum.
- **As cores dos níveis** foram escurecidas (`--bronze`, `--prata`, `--ouro`).
  Os tons claros das outras duas páginas somem sobre branco.
- **A borda luminosa do formulário** foi trocada por uma fita verde e âmbar no
  topo do cartão. Brilho não existe sobre branco.

---

## 5. O formulário

Cinco campos, todos obrigatórios:

| campo | validação |
|---|---|
| Seu nome completo | nome e sobrenome |
| Seu WhatsApp | DDD válido + 9 dígitos, o 9 na terceira posição |
| Sua chave Pix | CPF ou CNPJ com dígito verificador, telefone, e-mail ou chave aleatória |
| Nome de quem você indica | nome e sobrenome |
| WhatsApp de quem você indica | mesma regra, e não pode ser igual ao seu |

O envio espera a requisição sair antes de mostrar sucesso (`await`), ao
contrário das LPs de captura, que disparam e redirecionam. Como a pessoa
**fica na página**, mostrar "enviado" sem ter enviado seria mentira visível na
próxima indicação.

Na tela de sucesso, **"Indicar outra pessoa"** limpa apenas os dois campos do
indicado e devolve o formulário com nome, WhatsApp e chave Pix ainda
preenchidos.

---

## 6. A seção "O que você está indicando"

Os oito cursos e o posicionamento vieram da `captura-unicive`, nada foi
inventado:

| o quê | de onde |
|---|---|
| os 8 cursos | `captura-unicive/index.html`, seção "Principais graduações" |
| "A graduação EAD do concurseiro" | a mesma página |
| "reconhecidas pelo MEC" | a mesma página |
| 3 fotos de formandos | `captura-unicive/public/formados` (34 KB as três) |
| logo da marca | `captura-unicive/public/formados/LogoUnicive.png` |

As fotos foram **copiadas**, não referenciadas do outro deploy: a página não
pode quebrar se aquele projeto mudar de rota.

A ordem dos cursos não é a do material original. Serviços Jurídicos, Gestão da
Atividade Policial e Criminologia vêm primeiro porque são os que falam direto
com concurso, que é a dor do público desta página. Marketing Digital fecha.

---

## 7. Uma ambiguidade a confirmar

O material do programa diz:

> Nível 1 — Bronze (1 matrícula confirmada): R$ 50,00
> Nível 2 — Prata (2 matrículas confirmadas): R$ 100,00
> Nível 3 — Ouro (3 matrículas confirmadas): R$ 150,00
> A premiação é por quantidade de matrículas confirmadas (1ª, 2ª, 3ª).

Isso comporta duas leituras: ou cada matrícula paga o valor do seu degrau (e
três indicações somam R$ 300), ou chegar ao nível N paga aquele valor e só.

A página **não decide por nenhuma das duas**. Cada card mostra "1ª / 2ª / 3ª
matrícula confirmada" com o seu valor, exatamente como está no material. Se a
regra for acumulativa, vale acrescentar isso: "três indicações fecham R$ 300" é
um argumento forte e hoje não está sendo usado.

---

## 8. O que NÃO está na página

O **procedimento de pagamento** (enviar a chave Pix ao gestor financeiro, o
gestor realiza o pagamento) ficou de fora: é fluxo interno do time, não
informação para quem indica. Para o visitante, o que importa é que o Pix cai na
chave dele quando a matrícula for confirmada, e isso está dito.

---

## 9. Rastreamento

GTM server-side (`sgtm.cppem.com.br`), igual às outras páginas. Eventos no
`dataLayer`, todos com `bu: "UNICIVE"`:

| evento | quando |
|---|---|
| `indicacao_enviada` | envio aceito, com `indicacao_numero` |
| `indicacao_erro` | a requisição não saiu |
| `indicacao_nova_tentativa` | clique em "Indicar outra pessoa" |

Não há evento de `Lead` aqui: indicação não é lead de venda e contaria como
conversão nas campanhas.

---

## 10. Copy

Nenhum travessão em texto visível ao público, nas três páginas. É o sinal mais
reconhecível de texto gerado por IA, e uma landing de marca que pareça escrita
por IA perde credibilidade justamente com quem precisa confiar.

Para conferir depois de mexer na copy, o script
`achar-travessao.py` do scratchpad varre `index.html` e `script.js` ignorando
`<style>`, `<script>` e comentários.

---

## O banner do hero

O hero usa o **mesmo `background1.webp` do `captura-unicive`**, para as duas
páginas da UniCive abrirem com a mesma cara.

Duas consequências que o arquivo trouxe:

- A marca d'água do logo que existia no hero **saiu**. A arte já traz o símbolo
  da UniCive à esquerda, e seriam dois logos no mesmo lugar.
- O texto do hero virou **branco**, com o âmbar nos destaques. Por cima da arte
  há um véu verde que escurece o lado esquerdo, onde mora o texto, e vai
  ficando transparente à direita para a ilustração aparecer atrás do
  formulário. Sem o véu, o centro claro da imagem deixaria o texto ilegível.

No empilhado o texto ocupa a largura toda, então o véu deixa de ser lateral e
passa a cobrir por igual.

O resto da página continua claro: o hero é a única faixa verde.

---

## O carrossel

As três páginas usam o mesmo carrossel infinito, com a mesma classe
`.carrossel`. Uma correção feita numa transfere para as outras.

Como funciona: duas cópias da mesma fita, lado a lado, e a animação arrasta o
conjunto para a esquerda. Quando a primeira cópia acaba de sair, a segunda
está exatamente onde a primeira começou, então o salto de volta ao início não
se vê.

O deslocamento é **`-50% - metade do gap`**, não `-50%` puro. Com duas cópias
separadas por um gap, metade da largura total cai no meio desse espaço, e a
emenda daria um solavanco a cada volta. Medido no navegador nas três páginas,
o deslocamento do CSS bate com a largura de uma cópia inteira na casa do
centésimo de pixel.

Outros detalhes que não são óbvios:

- A segunda cópia é `aria-hidden`. Sem isso, um leitor de tela leria a mesma
  lista duas vezes.
- As fotos individuais têm `alt=""` e quem carrega a descrição é o contêiner,
  com `role="img"` e um `aria-label` do conjunto. São várias fotos da mesma
  cena; descrever uma a uma só encheria o leitor de tela de repetição.
- A segunda cópia usa os **mesmos `src`**, então o navegador não baixa nada de
  novo: 131 KB as 11 fotos no total.
- Passar o mouse pausa. `:focus-within` também, para quem navega por teclado.
- Com "reduzir movimento" ligado no sistema, a animação some, a fita vira uma
  faixa rolável com scroll-snap e a cópia duplicada é escondida.

A velocidade fica no `style="--duracao:"` do próprio elemento, para cada página
ajustar sem mexer no CSS: 50s aqui, proporcional à quantidade de fotos.
