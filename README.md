# Desafio Dio - Criando o Seu Blog Pessoal Com Next.js



## Configurando seu projeto Next.js



1. #### **Criando um novo projeto Next.js**

Comece criando um novo projeto Next.js. Se você ainda não instalou o Node.js e o npm (Node Package Manager), você precisará deles.

```javascript
yarn create next-app
```



Em seguida, serão solicitados os seguintes prompts (responda conforme o indicado):

```javascript
What is your project named?  my-static-blog
Would you like to use TypeScript?  Yes
Would you like to use ESLint?  Yes
Would you like to use Tailwind CSS?  Yes
Would you like to use `src/` directory?  Yes
Would you like to use App Router? (recommended)  Yes
Would you like to customize the default import alias? No / Yes
```



Depois de responder às solicitações, um novo projeto será criado com a configuração correta de acordo com suas respostas.

1. #### **Navegando até o diretório do projeto**

Vá para o diretório do projeto recém-criado:

```javascript
cd my-static-blog
```



## Estruturando o conteúdo do seu blog



1. #### **Criando um diretório "posts"**

Na raiz do seu projeto, crie um diretório chamado “posts”. É aqui que você armazenará seus arquivos Markdown, cada um representando uma postagem no blog.



1. #### **Escrevendo postagens de blog Markdown**

Dentro do diretório “posts”, crie arquivos Markdown para cada postagem do blog. Uma estrutura simples para um arquivo Markdown pode ser assim:

```javascript
---
título: "Título da postagem do seu blog"
subtítulo: "Subtítulo da postagem do seu blog"
data: "2024-04-30"
polegar: "https://Mario.dev/images/blog-nextjs-thumb.png"
---

# Título da postagem do seu blog

Seu conteúdo vai aqui.
```



A seção entre as linhas “---” é chamada frontmatter, onde você pode definir metadados para sua postagem no blog. Salve o arquivo com o nome que desejar, mas com a extensão ".md" (por exemplo, `first-article.md`).



## Renderizando conteúdo Markdown com Next.js

1. #### **Analisando Markdown**

Para renderizar o conteúdo Markdown, usaremos os pacotes [markdown-to-jsx](https://www.npmjs.com/package/markdown-to-jsx), [gray-matter](https://www.npmjs.com/package/gray-matter) e [react-syntax-highlighter](https://www.npmjs.com/package /react-syntax-highlighter). Instale-os em seu projeto:

```javascript
yarn add markdown-to-jsx gray-matter react-syntax-highlighter
```



Além disso, você precisa adicionar as seguintes dependências de desenvolvimento:

```javascript
yarn add -D @tailwindcss/typography @types/react-syntax-highlighter
```



1. #### **Criando uma página de listagem de postagens do blog**

Na pasta do aplicativo, crie um diretório `posts` com um arquivo `page.tsx`. Você pode usar o código a seguir para ler os arquivos markdown existentes.



```javascript
import fs from "fs";
import matter from "gray-matter";

import PostPreview from "./PostPreview";

const getPostMetadata = () => {
  const dir = `posts/`;

  const files = fs.readdirSync(dir);
  const markdownPosts = files.filter((file) => file.endsWith(".md"));

  const posts = markdownPosts.map((filename) => {
    const fileContents = fs.readFileSync(`posts/${filename}`, "utf-8");
    const matterResult = matter(fileContents);

    return {
      title: matterResult.data.title,
      date: matterResult.data.date,
      subtitle: matterResult.data.subtitle,
      thumb: matterResult.data.thumb,
      slug: filename.replace(".md", ""),
    };
  });

  return posts;
};

export const generateStaticParams = () => {
  return [{ locale: "en-us" }, { locale: "pt-br" }];
};

export const metadata: Metadata = {
  title:  "Posts Page",
  description: "Posts Page Description",
}

const PostPage = () => {
  const postMetadata = getPostMetadata();
  const postPreviews = postMetadata.map((post) => <PostPreview key={post.slug} {...post} />);

  return (
    <section className="px-6 max-w-full md:max-w-4xl lg:max-w-7xl pt-6">
      <h1 className="text-2xl font-bold">Blog posts</h1>
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-x-12">{postPreviews}</div>
    </section>
  );
};

export default PostPage;
```



Esta página é processada no lado do servidor e lê todos os arquivos `.md` presentes no diretório `posts/`. As informações de metadados são fornecidas para melhorar o SEO da página, e a função `generateStaticParams` é fornecida para forçar o Next.js a usar SSG para essa página.



O pacote `gray-matter` é usado para ler os metadados dos arquivos e mostrá-los usando o componente `PostPreview` (que deve ser salvo no diretório `posts/` como `PostPreview.tsx`).



```javascript
import Image from "next/image";
import Link from "next/link";

interface PostPreviewProps {
  slug: string;
  title: string;
  subtitle: string;
  date: string;
  thumb: string;
}

const PostPreview = (post: PostPreviewProps) => {
  return (
    <article className="my-8">
      {post.thumb && (
        <Link href={`/posts/${post.slug}`}>
          <figure className="w-full bg-slate-100 rounded-md px-5 py-8 my-6">
            <Image
              src={post.thumb}
              alt={post.title}
              width={500}
              height={500}
              className="w-7/12 max-h-40 object-contain mx-auto"
            />
          </figure>
        </Link>
      )}

      <p className="text-sm text-slate-400">{post.date}</p>

      <Link href={`/posts/${post.slug}`}>
        <h2 className="font-bold hover:underline text-lg mb-2">{post.title}</h2>
      </Link>
      <p className="text-slate-700 mb-4">{post.subtitle}</p>
    </article>
  );
};

export default PostPreview;
```



O componente `PostPreview` recebe as propriedades da postagem e renderiza um link para a postagem do blog. Uma imagem é opcionalmente renderizada, bem como o título, subtítulo e data da postagem.



1. #### **Criando um componente para renderizar Markdown**

No diretório `posts`, crie um novo subdiretório `[slug]` com um arquivo `page.tsx`. Neste arquivo, precisamos configurar uma página para renderizar o markdown:



```javascript
import Link from "next/link";
import fs from "fs";
import Markdown from "markdown-to-jsx";
import matter from "gray-matter";

import PreBlock from "./PreBlock";

const getPostContent = (slug: string) => {
  const file = `posts/${slug}.md`;
  const content = fs.readFileSync(file, "utf-8");
  const matterResult = matter(content);
  return { content: matterResult.content, header: matterResult.data };
};

export const generateStaticParams = () => {
  return [
    { slug: "my-blog-post" },
  ];
};

export async function generateMetadata({
  params: { locale, slug },
}: {
  params: { locale: string; slug: string };
}) {
  const { header } = getPostContent(slug, locale);

  return {
    title: header.title,
    description: header.subtitle,
    openGraph: {
      images: [
        {
          url: header.thumb,
          width: 800,
          height: 600,
        },
      ],
    },
  };
}

const PostPage = ({ params }: { params: { slug: string } }) => {
  const slug = params.slug;

  const { content, header } = getPostContent(slug);
  const publishedDate = header.date;

  return (
    <>
      {header.thumb && (
        <div
          className={`bg-slate-200 px-8 md:px-16 h-52 w-full bg-fixed bg-contain bg-no-repeat`}
          style={{
            backgroundImage: `url('${header.thumb}')`,
            backgroundPosition: "center 90px",
            backgroundSize: "auto 180px",
          }}
        />
      )}

      <div className="px-6 max-w-full md:max-w-4xl lg:max-w-7xl pt-6">
        <div className="flex justify-start w-full mb-6 hover:underline">
          <Link href="/posts" className="font-bold hover:underline mt-6">
            &lt;&lt; Back to Posts
          </Link>
        </div>

        <h1 className="text-3xl md:text-4xl font-bold">{header.title}</h1>
        <h2 className="text-xl my-1">{header.subtitle}</h2>
        <h2 className="pb-10">{publishedDate}</h2>

        <article className="prose lg:prose-lg">
          <Markdown
            options={{
              overrides: {
                pre: {
                  component: PreBlock,
                },
                img: {
                  props: {
                    className: "max-w-full mx-auto mt-6 mb-2",
                  },
                },
              },
            }}
          >
            {content}
          </Markdown>
        </article>
      </div>
    </>
  );
};

export default PostPage;
```



O pacote `gray-matter` é usado para ler os detalhes dos metadados de um arquivo específico. O nome do arquivo markdown é determinado pelo slug da URL. Por padrão, esta página processa o markdown dinamicamente do lado do servidor. Para tornar esse processo mais eficiente, utilizamos o SSG para pré-construir a página. Para fazer isso, precisamos incluir o slug do artigo no objeto de retorno `generateStaticParams`. Com base nessas informações, a função `generateMetadata` é usada para fornecer algumas informações de metadados dinâmicos para a página, melhorando o SEO da página.



A tag `Markdown` (de `markdown-to-jsx`) permite personalizar o estilo de saída do artigo. No código acima, a tag `pre` é substituída pelo componente `PreBlock`, melhorando o estilo do código-fonte (incluindo um marcador) nos artigos. Portanto, salve o seguinte código em `posts/[slug]/PreBlock.tsx`:



```javascript
"use client";

import { Prism as SyntaxHighlighter } from "react-syntax-highlighter";
import { materialDark as CodeStyle } from "react-syntax-highlighter/dist/esm/styles/prism";

const CodeBlock = ({ className, children }: { className: string; children: string }) => {
  let lang = "text"; // default monospaced text
  if (className && className.startsWith("lang-")) {
    lang = className.replace("lang-", "");
  }
  return (
    <SyntaxHighlighter language="javascript" style={CodeStyle}>
      {children}
    </SyntaxHighlighter>
  );
};

// markdown-to-jsx uses <pre><code/></pre> for code blocks.
const PreBlock = ({ children, ...rest }: { children: JSX.Element }) => {
  if ("type" in children && children["type"] === "code") {
    return CodeBlock(children["props"]);
  }
  return <pre {...rest}>{children}</pre>;
};

export default PreBlock;
```



#####  também precisa adicionar o pacote `@tailwindcss/typography` ao seu `tailwind.config.js`, como segue:

```javascript
/** @type {import('tailwindcss').Config} */

module.exports = {
  ...,
  plugins: [
    require('@tailwindcss/typography'),
  ],
}
```



O plugin oficial "Tailwind CSS Typography" fornece um conjunto de classes que você pode usar para adicionar belos padrões tipográficos a qualquer HTML básico que você não controla, como HTML renderizado de Markdown ou extraído de um CMS.

Finalmente, para garantir que seu projeto será capaz de acessar a miniatura do blog em https://rafaelf.dev/, você precisa adicionar isto ao seu arquivo `next.config.js`:

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
    ...,
    images: {
        domains: ['rafaelf.dev'],
    },
}

module.exports = nextConfig
```



## Executando seu blog estático

Com tudo configurado, agora você pode executar seu blog localmente:

```javascript
yarn dev
```

Visite http://localhost:3000/posts no seu navegador para ver seu blog em ação. Seu primeiro artigo será listado com um link apontando para o artigo, com o mesmo slug do nome do arquivo markdown - mas sem a extensão `.md` (ou seja, http://localhost:3000/posts/first-article).



## Conclusões

A criação de um blog usando arquivos Markdown e Next.js combina a simplicidade do Markdown com o poder de uma estrutura web moderna. Essa abordagem resulta em um blog rápido e otimizado para SEO, fácil de gerenciar e manter. Seguindo este tutorial, você adquiriu o conhecimento necessário para criar seu próprio blog e pode aprimorá-lo ainda mais com recursos e personalizações adicionais. Feliz blogging!



## Ou Segunada Opção



### Passos para criar seu blog pessoal com Next.js:



#### 1 - Crie um novo projeto Next.js usando o seguinte comando:

```plaintext
npx create-next-app blog
```



#### 2 - Acesse a pasta do projeto e instale as dependências necessárias:

```plaintext
npm install
```



#### 3 - Crie a estrutura do seu blog. O mais simples é usar o template de blog que o Next.js fornece. Você pode fazer isso usando o seguinte comando:



```plaintext
npx next init blog --example blog-starter
```



#### 4 - Acesse a pasta `blog/pages/blog` e crie o arquivo `index.js`. Este arquivo será a página inicial do seu blog. Você pode adicionar o seguinte código ao arquivo `index.js`:



```plaintext
import { useState } from "react";
import Link from "next/link";

export default function Home() {
  const [posts, setPosts] = useState([]);

  // Get the posts from the database

  // Render the posts

  return (
    <div>
      <h1>Meu Blog</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id}>
            <Link href="/posts/[id]" as={`/posts/${post.id}`}>
              <a>{post.title}</a>
            </Link>
          </li>
        ))}
      </ul>
    </div>
  );
}
```



#### 5 - Acesse a pasta `blog/pages/posts` e crie o arquivo `[id].js`. Este arquivo será a página de um post específico. Você pode adicionar o seguinte código ao arquivo `[id].js`:



```plaintext
import { useState, useEffect } from "react";
import Post from "next/post";

export default function PostPage({ post }) {
  const [comments, setComments] = useState([]);

  // Get the comments from the database

  // Render the comments

  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
      <ul>
        {comments.map((comment) => (
          <li key={comment.id}>
            <p>{comment.author}</p>
            <p>{comment.body}</p>
          </li>
        ))}
      </ul>
    </div>
  );
}
```



##### 6 - Agora você pode começar a criar posts e comentários para o seu blog. Você pode usar o comando `next dev` para iniciar o servidor de desenvolvimento e acessar seu blog no endereço `http://localhost:3000/`.

