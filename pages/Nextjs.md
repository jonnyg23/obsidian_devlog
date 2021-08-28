filters:: {"todo" true, "doing" true}

## A [[Javascript]] Framework for user-friendly static websites.
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
- **Content Animation**:
	- [[Framer Motion]]:
	  collapsed:: true
		- ### Info
			- This library is a combination of two APIs (the Framer API & Motion API)
			- Provides helpers for advanced physics-based animation.
			- Motion API generates automatic animations. You only need to set up the correct settings values.
			- Supports SSR & gesture recognizers like `hover`, `tap`, `pan`, and `drag`.
			- Easy to manipulate and alter colors.
			- **Easy to Learn**
			- Supports TypeScript
		- ### Installation
			-
			  ```bash 
			  yarn add framer-motion
			  # or run
			  npm install framer-motion
			  ```
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
	- [[React Transition Group]]:
	- [[React-Motion]]:
		- ### Info
			- It mainly provides 5 components: `Motion`, `spring`, `StaggeredMotion`, `TransitionMotion`, and `presets`.
			- `spring`
	- [[React Move]]:
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