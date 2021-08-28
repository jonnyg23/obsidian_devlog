## A [[Javascript]] Framework for user-friendly static websites.
- **Folder Structure**:
	- Basic folder structure of a Nextjs project [medium article](https://medium.com/@pablo.delvalle.cr/an-opinionated-basic-next-js-files-and-directories-structure-88fefa2aa759).
- **SEO with Nextjs**:
	- Setting SEO tags in the *Head* component
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