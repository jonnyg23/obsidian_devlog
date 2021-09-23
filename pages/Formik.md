-
  > Can be integrated with [[Material-ui]] in [[Nextjs]] to create [[forms]]
## Docs
	- [Formik docs for setting up with Material-UI] (https://formik.org/docs/examples/with-material-ui)
## Nextjs-MaterialUI Examples
	- **Example use for simple Email/Password form & Submit button with [[Yup]] validation**
		-
		  ```jsx
		  import React from 'react';
		  import ReactDOM from 'react-dom';
		  import { useFormik } from 'formik';
		  import * as yup from 'yup';
		  import Button from '@material-ui/core/Button';
		  import TextField from '@material-ui/core/TextField';
		  
		  const validationSchema = yup.object({
		    email: yup
		      .string('Enter your email')
		      .email('Enter a valid email')
		      .required('Email is required'),
		    password: yup
		      .string('Enter your password')
		      .min(8, 'Password should be of minimum 8 characters length')
		      .required('Password is required'),
		  });
		  
		  const WithMaterialUI = () => {
		    const formik = useFormik({
		      initialValues: {
		        email: 'foobar@example.com',
		        password: 'foobar',
		      },
		      validationSchema: validationSchema,
		      onSubmit: (values) => {
		        alert(JSON.stringify(values, null, 2));
		      },
		    });
		  
		    return (
		  
		  )
		  ```