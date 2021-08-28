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