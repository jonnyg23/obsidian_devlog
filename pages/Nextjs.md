filters:: {"todo" true, "doing" true}

## A [[Javascript]] Framework for user-friendly static websites.
	- ### Helps developer create static generation [[SSG]] and server-side rendering [[SSR]] websites.
## Starting New Project
	-
	  collapsed:: true
	  1. Init project with yarn
		-
		  ```bash
		  # Ensure latest version of yarn
		  yarn -version
		  
		  # Initialize New Next App
		  yarn create next-app
		  
		  # or for typescript
		  yarn create next-app --typescript
		  ```
	-
	  collapsed:: true
	  2. Init [[Docker]] with Next App
		- 2a. Add `Dockerfile` to root directory
			-
			  ```docker
			  # Install dependencies only when needed
			  FROM node:14-alpine AS deps
			  # Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
			  RUN apk add --no-cache libc6-compat
			  WORKDIR /app
			  COPY package.json yarn.lock ./
			  RUN yarn install --frozen-lockfile
			  
			  # Rebuild the source code only when needed
			  FROM node:14-alpine AS builder
			  WORKDIR /app
			  COPY . .
			  COPY --from=deps /app/node_modules ./node_modules
			  RUN yarn build
			  
			  # Production image, copy all the files and run next
			  FROM node:14-alpine AS runner
			  WORKDIR /app
			  
			  ENV NODE_ENV production
			  
			  RUN addgroup -g 1001 -S nodejs
			  RUN adduser -S nextjs -u 1001
			  
			  # You only need to copy next.config.js if you are NOT using the default configuration
			  # COPY --from=builder /app/next.config.js ./
			  COPY --from=builder /app/public ./public
			  COPY --from=builder --chown=nextjs:nodejs /app/.next ./.next
			  COPY --from=builder /app/node_modules ./node_modules
			  COPY --from=builder /app/package.json ./package.json
			  
			  USER nextjs
			  
			  EXPOSE 3000
			  
			  # Next.js collects completely anonymous telemetry data about general usage.
			  # Learn more here: https://nextjs.org/telemetry
			  # Uncomment the following line in case you want to disable telemetry.
			  ENV NEXT_TELEMETRY_DISABLED 1
			  
			  CMD ["yarn", "start"]
			  ```
		- 2b. Add `.dockerignore` to root directory
			-
			  ```docker
			  Dockerfile
			  .dockerignore
			  node_modules
			  npm-debug.log
			  README.md
			  .next
			  ```
	-
	  3. Building, Running, & Stopping Docker image
		- 3a. **Build** Docker Image
			-
			  ```bash
			  docker build -t NAME_OF_APP .
			  ```
		- 3b. **Run** Docker Image
			-
			  ```bash
			  docker run -p 3000:3000 NAME_OF_APP
			  ```
		- 3c. **View** Dockerized Next App locally at `localhost:3000`
		- 3d. **Stopping** Docker Image
			-
			  1. Execute Command `docker ps` in terminal
			-
			  2. Copy the container id
			-
			  3. Execute command `docker stop container id` in terminal
	-
	  4. Adding `PORT` to `package.json`
		- In the `scripts` object, at this to `start`:
			-
			  ```json
			  "start": "next start -p ${PORT:=3000}"
			  ```
	-
### Vercel Nextjs Examples
	- If you need any assistance with integrating other softwares or just to see design patterns with Nextjs, go to the vercel nextjs Github examples link [here](https://github.com/vercel/next.js/tree/canary/examples)
- **Folder Structure**:
	- Basic folder structure of a Nextjs project [medium article](https://medium.com/@pablo.delvalle.cr/an-opinionated-basic-next-js-files-and-directories-structure-88fefa2aa759).
- **SEO**:
	- Setting SEO tags in the *Head* component
	  collapsed:: true
	  Nextjs gives us a `Head` tag where we can specify page title, description, open graph tag, and icon links. This is inserted as such:
	  ```jsx
	  import Head from 'next/head';
	  import Image from 'next/image';
	  import styles from '../styles/Home.module.css';
	  
	  export default function Home() {
	    return (
	      <div className={styles.container}>
	        <Head>
	          <title>My Clothing Store</title>
	          <meta name="description" content="Buy things at my store!" />
	          <meta property="og:title" content="My Clothing Store" />
	          <link rel="icon" href="/favicon.ico" />
	        </Head>
	        <main></main>
	        <footer></footer>
	      </div>
	    )
	  }
	  ```
	- Dynamic SEO Meta Tags [Tutorial](https://www.youtube.com/watch?v=8BrZeaw3sLQ&t=163s)
	  collapsed:: true
		- ### This is a sample productsID page. 
		  The folder structure is:
		- `pages`
			- `products`
				- `[productId].js`
			- `_app.js`
			  `_document.js`
			  `index.js`
		-
		- This is a look into the `[productId].js` file where the SEO meta tags are dynamically created:
		-
		  ```jsx
		  import Head from 'next/head';
		  import styles from '../../styles/Home.module.css';
		  
		  export default function Product({ productId, title }) {
		    return (
		      <div className={styles.container}>
		        <Head>
		          <title> { title } - My Clothing Store</title>
		          <meta name="description" content={`Learn more about ${title}`} />
		          <meta property="og:title" content={`${title} - My Clothing Store`} />
		          <link rel="icon" href="/favicon.ico" />
		        </Head>
		        <main className={styles.main}>
		          <h1 className={styles.title}>{title}</h1>
		          <p>Product ID: {productId}</p>
		        </main>
		      </div> 
		    )
		  }
		  
		  export async function getStaticProps({ params = {} } = {}) {
		    return {
		      props: {
		        productId: params.productId,
		        title: `Product ${params.productId}`
		      }
		    }
		  }
		  
		  export async function getStaticPaths() {
		    const paths = [...new Array(5)].map((i, index) => {
		      return {
		        params: {
		          productId: `${index + 1}`
		        }
		      };
		    });
		    return {
		      paths,
		      fallback: false,
		    };
		  }
		  ```
- **Dockerizing Nextjs**:
- **Authentication**:
	- [[Auth0]]:
	  collapsed:: true
		-
		  Common used functions from the nextjs-auth0 library:
		  ```js
		  import { useUser, withPageAuthRequired } from "@auth0/nextjs-auth0";
		  - const Profile = () => {
		    const { user, error, isLoading } = useUser();
		  - if (isLoading) return <div>Loading...</div>;
		    if (error) return <div>{error.message}</div>;
		    if (!user) return <Link href="/api/auth/login"><a>Login</a></Link>;
		    return <div>Hello {user.name}, <Link href="/api/auth/logout"><a>Logout</a></Link>< /link 
		  // Then wrap exported function as such
		  export default withPageAuthRequired(Profile);
		  ```
	- [[next-auth]]:
		- Nextjs Open Source Auth Library [Next-auth](https://next-auth.js.org/)
- **Styling**:
  collapsed:: true
	- [[Material-ui]]:
		- Preventing Server side rendering [[SSR]] components
			- Import the `NoSsr` hook from material-ui as such:
			  ```jsx
			  import { NoSsr } from "@material-ui/core";
			  ```
			  And then wrap the component that you want to prevent from SSR with this tag:
			  ```jsx
			  <NoSsr>
			    {YOUR_COMPONENT}
			  </NoSsr>
			  ```
- **Content Animation Libraries**:
  collapsed:: true
	- [[Framer Motion]]:
		- ### Info
			- This library is a combination of two APIs (the Framer API & Motion API)
			- Provides helpers for advanced physics-based animation.
			- Motion API generates automatic animations. You only need to set up the correct settings values.
			- Supports SSR & gesture recognizers like `hover`, `tap`, `pan`, and `drag`.
			- Easy to manipulate and alter colors.
			- **Easy to Learn**
			- Supports TypeScript
		- ### Installation
		  collapsed:: true
			-
			  ```bash 
			  yarn add framer-motion
			  # or run
			  npm install framer-motion
			  ```
		- ### Examples
			- [vercel/nextjs Github](https://github.com/vercel/next.js/tree/canary/examples/with-framer-motion)
	- [[Remotion]]:
	  collapsed:: true
		- ### Info
			- Introduced at the beginning of 2021.
			- Allows you to create animations using common web technologies like HTML, CSS, JavaScript, TypeScript, etc.
			- No additional knowledge about video editing is required.
			- Remotion Player gives you the feeling of a real video editor.
			- Remotion Player can be used to play and review your video using your browser.
		- ### Installation
			- You should install `Node.js` and `FFmpeg` before using Remotion. Then you need to extract `FFmpeg` to any folder and set its path to system variables.
			- After installing the dependencies above, you can create your first Remotion project by running:
			  ```bash
			  yarn create video
			  # or run
			  npm init video
			  ```
	- [[React-Spring]]:
	  collapsed:: true
		- ### Info
			- A spring-physics-based animation library.
			- This library gives a very realistic spring-like motion to objects.
			- Provides Hooks for handling (`useTrail`, `useChain`, `useSpring`, `useTransition`, `useSprings`)
			- Testings is supported with Jest
			- **Beginner-friendly documentation**
			- Ability to add animations without relying on React rendering updates frame-by-frame.
			- Supports both web and [[React Native]]  applications.
		- ### Installation
			-
			  ```bash
			  yarn add react-spring
			  # or run
			  npm install react-spring
			  ```
	- [[React Transition Group]]:
	  collapsed:: true
		- ### Info
			- Simple components for defining animations, unlike react-spring.
			- Library does not define styles, but instead manipulates the DOM.
			- Very straightforward approach
			- Uses built in components such as `Transition` to set animations and transitions to elements, thereby separating your elements from your animations.
			- Supports TypeScript
		- ### Installation
			-
			  ```bash
			  yarn add react-transition-group
			  # or run
			  npm i react-transition-group
			  ```
	- [[React-Motion]]:
	  collapsed:: true
		- ### Info
			- It mainly provides 5 components: `Motion`, `spring`, `StaggeredMotion`, `TransitionMotion`, and `presets`.
			- **More complicated than other libraries**
			- `spring`
				- Helper function that guides how components animate
			- `Motion`
				- Components used to animate a component
			- `presets`
				- Predefined animation object
			- `TransitionMotion`
				- Component used for animating the mount and unmount of components
			- `StaggeredMotion`
				- A components used for animating components whose animations depend on each other.
		- ### Installation
			-
			  ```bash
			  yarn add react-motion
			  # or run
			  npm i react-motion
			  ```
	- [[React Move]]:
	  collapsed:: true
		- ### Info
			- Supports [[React]], [[React Native]], and [[React-VR]].
			- Has fine-grained control of delay, duration, and easing.
			- Supports [[TypeScript]]
			- **Simpler than React Motion**
		- ### Installation
			-
			  ```bash
			  npm install react-move
			  ```
	-
- **Analytics**:
	- [[Google Analytics]]:
		-