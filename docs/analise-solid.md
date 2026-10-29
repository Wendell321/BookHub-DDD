# Q3 — Inspeção dos princípios SOLID no BookHub

## 1. Metodologia de inspeção

A inspeção foi realizada por leitura estática dos arquivos-fonte disponíveis no pacote do BookHub, sem executar alterações no código. Foram selecionados módulos e componentes que concentram responsabilidades, decisões condicionais ou dependências concretas. Para cada ocorrência, foram registrados o arquivo, o trecho relevante, o princípio relacionado, a justificativa e uma sugestão de refatoração.

Para reproduzir a análise, outra pessoa pode:
1. descompactar os projetos de backend e frontend;
2. abrir os arquivos indicados abaixo;
3. localizar os trechos pelas linhas informadas (as linhas podem variar se os arquivos forem editados);
4. comparar as responsabilidades e dependências observadas com os princípios SOLID;
5. avaliar se a sugestão de refatoração reduziria acoplamento ou concentraria cada responsabilidade.

A análise é estática e aponta oportunidades de melhoria, não erros funcionais comprovados. Os princípios SOLID são diretrizes de design; a classificação depende do contexto e deve ser validada pela equipe.

## 2. Ocorrência 1 — SRP no formulário de publicação de livros

**Arquivo:** `BookHub-Frontend/bookhub-frontend/src/views/UploadBookView.vue`  
**Trechos:** linhas 35–73 e 75–109.

```js
function handleCoverChange(event) {
  const file = event.target.files[0]
  if (!file) return
  coverFile.value = file
  coverPreviewUrl.value = URL.createObjectURL(file)
}

function validate() {
  errors.title = ''
  errors.description = ''
  errors.cover = ''
  errors.file = ''
  // validação dos campos...
  return Object.values(errors).every((message) => message === '')
}

async function handleSubmit() {
  // validação, montagem do FormData, envio, progresso,
  // mensagens de sucesso/erro e navegação
}
```

**Princípio relacionado:** SRP — Princípio da Responsabilidade Única.

**Análise:** o componente coordena seleção e pré-visualização de arquivos, validação de campos, montagem do `FormData`, chamada à API, acompanhamento de progresso, mensagens e navegação. Mudanças nas regras de validação, no envio de arquivos ou no fluxo visual podem exigir alterações no mesmo componente.

**Refatoração sugerida:** extrair a validação para um módulo reutilizável e o tratamento de seleção/pré-visualização de arquivos para um composable. Manter no componente a coordenação da interface.

## 3. Ocorrência 2 — SRP na tela de cadastro

**Arquivo:** `BookHub-Frontend/bookhub-frontend/src/views/auth/RegisterView.vue`  
**Trechos:** linhas 30–50 e 53–74.

```js
function validate() {
  Object.keys(errors).forEach((key) => (errors[key] = ''))
  // valida nome, usuário, e-mail, senha e confirmação
  return Object.values(errors).every((message) => message === '')
}

async function handleSubmit() {
  apiErrorMessage.value = ''
  if (!validate()) return
  isSubmitting.value = true
  try {
    await register({
      username: form.username.trim(),
      email: form.email.trim(),
      password: form.password,
      confirmPassword: form.confirmPassword,
      fullName: form.fullName.trim(),
    })
    router.push({ name: 'login' })
  } catch (error) {
    apiErrorMessage.value = error.message
  } finally {
    isSubmitting.value = false
  }
}
```

**Princípio relacionado:** SRP.

**Análise:** a tela mantém estado do formulário, aplica regras de validação, prepara os dados, chama o serviço de cadastro, controla estado de envio, trata falhas e decide a navegação após o sucesso.

**Refatoração sugerida:** mover as regras de validação para um validador independente e deixar o componente focado na interação visual. O serviço de autenticação deve continuar responsável pela comunicação com a API.

## 4. Ocorrência 3 — SRP na tela de perfil

**Arquivo:** `BookHub-Frontend/bookhub-frontend/src/views/profile/MyProfileView.vue`  
**Trechos:** linhas 30–41, 43–65 e 67–91.

```js
function handleFileChange(event) {
  const file = event.target.files[0]
  if (!file) return
  selectedFile.value = file
  previewUrl.value = URL.createObjectURL(file)
}

function validate() {
  errors.fullName = ''
  errors.bio = ''
  // valida os campos do perfil
  return !errors.fullName && !errors.bio
}

async function handleSubmit() {
  // monta FormData, chama updateProfile,
  // atualiza a imagem e controla mensagens/estado
}
```

**Princípio relacionado:** SRP.

**Análise:** o mesmo componente carrega os dados do perfil, gerencia formulário, valida dados, cria prévia de imagem, envia atualização e controla mensagens de sucesso e erro.

**Refatoração sugerida:** separar a validação do perfil e o gerenciamento de pré-visualização de imagem em funções/composables próprios. Manter no componente a apresentação e a coordenação do formulário.

## 5. Ocorrência 4 — SRP no cliente HTTP

**Arquivo:** `BookHub-Frontend/bookhub-frontend/src/services/api.js`  
**Trechos:** linhas 14–20 e 22–55.

```js
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

api.interceptors.response.use(
  (response) => response.data,
  (error) => {
    if (error.response.status === 401) {
      localStorage.removeItem('token')
      localStorage.removeItem('user')
      window.location.href = '/login'
    }
    // normalização de erros HTTP, de rede e de preparação
  }
)
```

**Princípio relacionado:** SRP.

**Análise:** o módulo configura Axios, injeta credenciais, transforma respostas, normaliza erros e também decide encerrar a sessão e redirecionar a interface em caso de HTTP 401. A política de autenticação e navegação está acoplada ao transporte HTTP.

**Refatoração sugerida:** manter no cliente HTTP a configuração e a normalização de respostas; delegar o tratamento de sessão expirada a um serviço ou mecanismo de autenticação dedicado, evitando redirecionamento diretamente no interceptor.

## 6. Ocorrência 5 — SRP no middleware de erros

**Arquivo:** `BookHub-Backend/bookhub-api/middlewares/errorHandler.js`  
**Trechos:** linhas 4–35 e 37–51.

```js
function removeUploadedFiles(req) {
    // identifica req.file/req.files
    // remove arquivos do disco com fs.unlink(...)
}

module.exports = (err, req, res, next) => {
    console.error(err)
    removeUploadedFiles(req)
    const statusCode = err.status || 500
    const errors = err.errors || []
    return error(res, err.message || 'Ocorreu um erro inesperado.',
        statusCode, errors)
}
```

**Princípio relacionado:** SRP.

**Análise:** o middleware registra o erro, limpa arquivos enviados que podem ter ficado órfãos, determina o status HTTP, extrai erros de validação e formata a resposta. A limpeza de arquivos é uma responsabilidade de armazenamento diferente da tradução de erros HTTP.

**Refatoração sugerida:** extrair a limpeza de uploads para um serviço dedicado e fazer o middleware de erros coordenar o tratamento e a resposta. O serviço de limpeza pode ser chamado pelo middleware sem que toda a lógica de arquivos fique nele.

## 7. Ocorrência 6 — DIP no middleware de autenticação

**Arquivo:** `BookHub-Backend/bookhub-api/middlewares/auth.js`  
**Trechos:** linhas 1 e 14–21.

```js
const { verifyToken } = require('../config/jwt');

// ...
const decoded = verifyToken(token)
req.user = decoded
next()
```

**Princípio relacionado:** DIP — Princípio da Inversão de Dependência.

**Análise:** o middleware de alto nível, responsável pela política de autenticação HTTP, importa diretamente uma implementação concreta de verificação de JWT. Isso dificulta substituir o mecanismo de autenticação ou testar o middleware sem depender dessa implementação concreta.

**Refatoração sugerida:** receber uma função/porta de verificação de token por injeção de dependência ou por uma fábrica de middleware. A implementação JWT concreta seria configurada na composição da aplicação, e os testes poderiam fornecer uma implementação simulada.

**Observação:** a dependência concreta não é automaticamente um defeito; a sugestão é útil caso o projeto precise substituir ou simular o verificador de tokens.

## 8. Ocorrência 7 — OCP no filtro de upload

**Arquivo:** `BookHub-Backend/bookhub-api/middlewares/bookMulter.js`  
**Trechos:** linhas 16–34.

```js
function fileFilter(req, file, cb) {
    if (file.fieldname === 'cover') {
        const allowedTypes = ['image/jpeg', 'image/png', 'image/webp']
        // aceita ou rejeita a capa
    }

    if (file.fieldname === 'file') {
        const allowedTypes = ['application/pdf', 'application/epub+zip']
        // aceita ou rejeita o arquivo do livro
    }

    cb(new Error('Campo de arquivo não reconhecido.'))
}
```

**Princípio relacionado:** OCP — Princípio Aberto/Fechado.

**Análise:** a seleção do conjunto de formatos permitidos depende de condicionais fixas para os campos `cover` e `file`. Se o sistema passar a aceitar novos campos de upload ou novos grupos de formatos, será necessário alterar a função existente e seus ramos condicionais.

**Refatoração sugerida:** declarar as regras de campo e tipos permitidos em uma configuração ou mapa de validadores. Assim, novas regras podem ser adicionadas pela configuração, reduzindo a necessidade de modificar o fluxo principal do filtro.

## 9. Limites da inspeção

O pacote de backend fornecido contém arquivos que importam módulos em `modules/search`, `modules/user`, `modules/book`, `modules/comment` e `modules/Like`, mas essas implementações não estavam presentes no ZIP recebido. Por isso, esta análise não atribui violações a controllers, models ou serviços desses módulos ausentes. As ocorrências listadas se baseiam apenas em arquivos-fonte disponíveis nos pacotes fornecidos.

Também não foram incluídas violações de LSP ou ISP, pois os trechos inspecionados não fornecem evidência suficientemente clara de hierarquias de subtipos ou interfaces amplas. Não é adequado inventar uma ocorrência apenas para representar cada letra do acrônimo. A atividade permite que as seis ocorrências sejam distribuídas entre os princípios.

## 10. Referência conceitual

A avaliação utiliza as definições gerais de SOLID: SRP (uma responsabilidade principal), OCP (extensão sem alteração recorrente do fluxo existente) e DIP (dependência de abstrações em vez de implementações concretas). As classificações são uma análise do código do BookHub e as sugestões são propostas de refatoração, não alterações já implementadas.
