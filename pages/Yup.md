-
  >A JavaScript schema builder for value parsing and validation.
- [yup Github Docs](https://github.com/jquense/yup)
## Installation
	-
	  ```bash
	  npm install -S yup
	  # or
	  yarn add yup
	  ```
## Usage
	-
	  ```jsx
	  import * as yup from 'yup';
	  
	  let schema = yup.object().shape({
	    name: yup.string().required(),
	    age: yup.number().required().positive().integer(),
	    email: yup.string().email(),
	    website: yup.string().url(),
	    createdOn: yup.date().default(function () {
	      return new Date();
	    }),
	  });
	  
	  // check validity
	  schema
	    .isValid({
	      name: 'jimmy',
	      age: 24,
	    })
	    .then(function (valid) {
	      valid; // => true
	    });
	  
	  // you can try and type cast objects to the defined schema
	  schema.cast({
	    name: 'jimmy',
	    age: '24',
	    createdOn: '2014-09-23T19:25:25Z',
	  });
	  // => { name: 'jimmy', age: 24, createdOn: Date }
	  ```