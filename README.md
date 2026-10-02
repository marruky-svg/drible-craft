# Drible — Exercício Prático Craft CMS 5

Projeto base em Craft CMS 5 com uma homepage dinâmica: um slider e uma listagem de artigos geridos no backoffice, e uma página de detalhe para cada artigo.

## Requisitos

- [DDEV](https://ddev.com) (com um provedor Docker: OrbStack, Docker Desktop ou Colima)

Não é preciso ter PHP, Composer nem MySQL instalados localmente: tudo corre dentro dos containers do DDEV.

## Arranque

A base de dados e o ficheiro `.env` foram enviados à parte.

```bash
git clone https://github.com/marruky-svg/drible-craft.git
cd drible-craft

# 1. Colocar o .env enviado na raiz do projeto
# 2. Colocar o dump enviado em database/drible-craft.sql.gz

ddev start
ddev composer install
ddev import-db --file=database/drible-craft.sql.gz
ddev craft up
ddev launch
```

- Frontend: https://drible-craft.ddev.site
- Backoffice: https://drible-craft.ddev.site/admin (credenciais enviadas junto com o `.env`)

O `ddev craft up` aplica migrações e alterações pendentes da Project Config. Num arranque limpo não deve haver nada a aplicar, mas garante que a base de dados e os ficheiros em `config/project/` ficam sincronizados.

## Estrutura

```
config/project/        Schema do backoffice (secções, entry types, campos, volume) em YAML
templates/
  _layout.twig         Layout base: HTML, cabeçalho, rodapé e blocks
  index.twig           Homepage: slider e os 3 artigos mais recentes
  artigos/_entry.twig  Página de detalhe de um artigo
web/css/main.css       Estilos
web/uploads/           Imagens de teste carregadas no backoffice
```

## Modelo de conteúdo

| Secção   | Tipo    | Entry type | Campos                                              | URL               |
|----------|---------|------------|-----------------------------------------------------|-------------------|
| Articles | Channel | Article    | Title, Summary, Image                               | `artigos/{slug}`  |
| Slider   | Channel | Slide      | Title, Image, Supporting text (opcional), Link (opcional) | Sem URL     |

## Decisões técnicas

**Craft 5 em vez da terminologia do enunciado.** O Craft 5 removeu os grupos de campos: os campos são globais e são associados às secções através de entry types. Os "grupos de campos" pedidos correspondem aqui aos entry types `Article` e `Slide`. O título usa o campo nativo Title, em vez de um campo de texto adicional.

**Campo Link em vez de texto simples.** O enunciado sugere um campo de texto para o link do slide. Usei o tipo Link do Craft 5, que valida o URL, permite apontar para uma entrada interna (o link mantém-se correto se o slug mudar) e inclui um rótulo para o texto do botão.

**Campo de imagem partilhado.** O mesmo campo `image` é usado nos artigos e nos slides, com rótulos diferentes em cada layout, porque as regras de validação são iguais.

**Eager loading.** As queries usam `.with(['image'])` para carregar as imagens de todas as entradas numa só query, evitando o problema N+1.

**Image transforms.** As imagens dos cartões são servidas redimensionadas (800×500) em vez do original. As versões geradas ficam fora do Git, porque são regeneradas automaticamente.

**Slider.** Swiper 11 via CDN, para não exigir um passo de build. Usa `rewind` em vez de `loop` (funciona com qualquer número de slides) e não tem autoplay, que exigiria um controlo de pausa por razões de acessibilidade. A transição é desativada quando o sistema pede movimento reduzido. Com um único slide, a navegação não é gerada.

**Campos opcionais.** O texto de apoio e o botão só são gerados quando têm conteúdo. Um dos slides de teste foi deixado sem eles de propósito.

**Uploads no repositório.** Num projeto real, `web/uploads` ficaria fora do Git e seria sincronizado à parte. Aqui está incluído para que as imagens correspondam ao dump enviado e o projeto arranque sem passos extra.

**Ordem do slider.** O enunciado pede uma secção do tipo Channel, que ordena por data de publicação. Num projeto para cliente, consideraria o tipo Structure, que permite ao editor reordenar os slides arrastando-os.
