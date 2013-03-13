HSP API Server
===============

What is it?
----------

Provides a public facing Restful API for the Hosted Services Portal. The functions provided by this API should
be specific to account management.  Separate API's will be available for SpamStopsHere, RestorEmail, etc settings.

Where is it?
----------

`https://api.greenviewdata.com`

Version 1 - Resources Available
----------

* Clients
    * GET /v1/clients
    * POST /v1/clients
    * GET /v1/clients/`id`
    * PUT /v1/clients/`id`
    * DELETE /v1/clients/`id`
* Client by Domain Name
    * GET /v1/domains/`domain_name`/client
  	* PUT /v1/domains/`domain_name`/client
		* DELETE /v1/domains/`domain_name`/client
* Auth Tokens
    * GET /v1/clients/`id`/auth_token
* Domains
    * GET /v1/domains/`domain_name`
    * PUT /v1/domains/`domain_name`
    * GET /v1/domains/`domain_name`/status
* Packages
    * GET /v1/packages/by_tag/`tag_name`
* Resellers
    * GET /v1/resellers/new
    * POST /v1/resellers

---

Authentication
----------

You must have a valid Reseller API token and API key. Use basic auth with the token as the user and the key as the password.

Resource Details
----------

### Get Clients

* `GET /v1/clients` will return all of the reseller's clients

  ```json
  [
	  {
			"id": 49632,
			"package_uid": null,
			"user_count": 1,
			"email_address": "phil@greenviewdata.com",
			"name": "Test Customer",
			"domains": [
				{
					"name": "customer-domain.test",
					"mx_records": [
						{
							"priority": 10,
							"hostname": "customer-domain-test.relay1g.spamh.com."
						},
						{
							"priority": 20,
							"hostname": "customer-domain-test.relay1h.spamh.com."
						}
					],
					"mail_servers": [
						"mail.customer-domain.test"
					]
				}
			]
		}
  ]
  ```

### Create (POST) Client

* `POST /v1/clients`

  ```json
	{
		"package_uid": "spamstopshere-standard",
		"user_count": 1,
		"email_address": "phil@greenviewdata.com",
		"name": "Test Customer",
		"domains": [
			{
				"name": "customer-domain.test",
				"mail_servers": [
					"mail.customer-domain.test"
				]
			}
		]
	}
  ```

  Success Response will be `201 Created` with the client returned as json in the same format as GET /clients/id:

  Error Response will be `422 Unprocessable Entity` with json:

  ```json
	{
		"errors": {
		  "package_uid": ["specific attribute error message"],
		  "base": ["non specific error message"]
		}
	}
  ```

	Required Fields:

    * package_uid
    * user_count
    * domains
        * name

### Get Client

* `GET /v1/clients/49632` will return information about a client found by its numeric id
* `GET /v1/domains/customer-domain.test/client` will return the client found by any of the client's domain names
	
  ```json
  {
		"id": 49632,
		"package_uid": null,
		"user_count": 1,
		"email_address": "phil@greenviewdata.com",
		"name": "Test Customer",
		"domains": [
			{
				"name": "customer-domain.test",
				"mx_records": [
					{
						"priority": 10,
						"hostname": "customer-domain-test.relay1g.spamh.com."
					},
					{
						"priority": 20,
						"hostname": "customer-domain-test.relay1h.spamh.com."
					}
				],
				"mail_servers": [
					"mail.customer-domain.test"
				]
			}
		]
	}
	```

### Update (PUT) Client

* `GET /v1/clients/49632` will update the client with id 49632
* `GET /v1/domains/customer-domain.test/client` will update the client containing the domain customer-domain.test
	
	IMPORTANT: If the domains array is passed, any domains not listed will be removed from the account.
	
  ```json
	{
		"package_uid": "spamstopshere-standard",
		"user_count": 1,
		"email_address": "phil@greenviewdata.com",
		"name": "Test Customer",
		"domains": [
			{
				"name": "customer-domain.test",
				"mail_servers": [
					"mail.customer-domain.test"
				]
			}
		]
	}
  ```
  
  Success Response will be `200` with the client returned as json in the same format as GET /clients/id:

  Error Response will be `422 Unprocessable Entity` with json:

  ```json
	{
		"errors": {
		  "package_uid": ["specific attribute error message"],
		  "base": ["non specific error message"]
		}
	}
  ```

	Required Fields:

    * package_uid
    * user_count
    * domains
        * name

### Delete Client

* `DELETE /v1/clients/11111111` will cancel this client with id 11111111
* `DELETE /v1/domains/customer-domain.test/client` will cancel the client having a domain name named customer-domain.test

	IMPORTANT: THIS WILL RESULT IN LOSS OF EMAIL IF THE CLIENT'S DOMAIN MX RECORDS ARE STILL POINTING TO THE SPAMSTOPSHERE SERVICE.
	
  Success Response will be `200`

  Error Response will be `422 Unprocessable Entity` with json:

  ```json
	{
		"errors": {
		  "base": ["Could not cancel account"]
		}
	}
  ```
	
### Get Domain Status

* `GET /v1/domains/vedit.com/status`

  Returns the current status of the domain. Possible values are:

    * initializing = domain is still being setup on our servers, don't change mx records yet
    * ready = domain is setup and we're ready to process email for it
    * unknown = some sort of error occurred figuring out the status

  ```json
	{
		"status": "initializing"
	}
  ```

### Update Domain

* `PUT /v1/domains/vedit.com`

  Update attributes on the domain. Currently, you may only update a domain's mail servers.
	
  ```json
	{
		"mail_servers": [
			"mail.customer-domain.test"
		]
	}
  ```
  
  Success Response will be `200` with the domain returned as json in the same format as `GET /v1/domains/vedit.com`:

  Error Response will be `422 Unprocessable Entity` with json:

  ```json
	{
		"errors": {
			"mail_servers": ["are invalid. Your subscription does not allow multiple mail servers, please upgrade to unlock this feature."]
		}
	}
  ```

### Get Auth Token to Login customer admin user

* `GET /v1/domains/vedit.com/auth_token`
* `GET /v1/clients/1234/auth_token`

	Returns a token that can be used to login as the admin user for a particular client. The token will expire within 1 minute of being issued.
	
	```json
	{
		"token": "secrettokensecrettokensecrettokensecrettoken",
		"protocol": "https",
		"login_hostname": "reseller-branded-url.test",
		"path": "/path/to/login/with/auth/token?query=args"
	}
	```

### Get Packages by Tag

* `GET /v1/packages/by_tag/cpanel`

	Returns an ordered array of packages.
	
  `added_features` will return a list of features not included in the previous package in the array.

  ```json
  [
		{
			"id": 1,
			"uid": "spamstopshere-standard",
			"name": "Standard",
			"description": "",
			"position": 1,
			"free_trial": 30,
			"features": [
				{
					"id": 1,
					"name": "SpamStopsHere Anti-Spam",
					"blurb": "",
					"description": "",
					"position": 1
				},
				{
					"id": 10,
					"name": "Brilliant 24/7/365 Support",
					"blurb": "800-458-3348",
					"description": "",
					"position": 12
				}
			],
			"added_features": [
				{
					"id": 1,
					"name": "SpamStopsHere Anti-Spam",
					"blurb": "",
					"description": "",
					"position": 1
				},
				{
					"id": 10,
					"name": "Brilliant 24/7/365 Support",
					"blurb": "800-458-3348",
					"description": "",
					"position": 12
				}
			],
			"reseller_cost": {
				"domain_cents": 50,
				"domain_formatted": "$0.50",
				"mailbox_cents": 100,
				"mailbox_formatted": "$1.00"
			}
		}
	]
  ```

### Create (POST) Reseller

Since this API call is open to the public, creating a reseller is a two step process. Reseller sign-ups are not meant to be fully automated. Human interaction at the end of the calling application is expected. Therefore, you must first initialize a new reseller to get a captcha question and id. Then post to create the new reseller with the captcha id and captcha answer.

* `GET /v1/resellers/new`

  ```json
	{
		"captcha": {
			"id": "abc123",
			"question": "If tomorrow is Saturday, what day is today?"
		},
		"reseller": {
			"name": null,
			"first_name": null,
			"last_name": null,
			"email_address": null,
			"address": null,
			"city": null,
			"state": null,
			"postal_code": null,
			"country": null,
			"phone": null,
			"username": null,
			"tags": []
		}
	}
	```

* `POST /v1/resellers`

  ```json
	{
		"captcha": {
			"id": "abc123",
			"answer": "friday"
		},
		"reseller": {
			"name": "Greenview Data",
			"first_name": "Phil",
			"last_name": "Green",
			"email_address": "phil@vedit.com",
			"address": "8178 Jackson Rd",
			"city": "Ann Arbor",
			"state": "Michigan",
			"postal_code": "48103",
			"country": "United States",
			"phone": "734-426-7500",
			"username": "my-reseller-account",
			"application_name": "cpanel"
		}
	}
  ```

  Success Response will be `201 Created` with the reseller returned as json:

	```json
	{
		"reseller": {
			"id": 182904,
			"name": "Greenview Data",
			"first_name": "Phil",
			"last_name": "Green",
			"email_address": "phil@vedit.com",
			"address": "8178 Jackson Rd",
			"city": "Ann Arbor",
			"state": "Michigan",
			"postal_code": "48103",
			"country": "United States",
			"phone": "734-426-7500",
			"username": "my-reseller-account",
			"application_name": "cpanel"
		},
		"captcha": {
			"id": "abc123",
			"question": "If tomorrow is Saturday, what day is today?"
		}
	}
	```
	
  Error Response will be `422 Unprocessable Entity` with json. A new captcha question and id will be generated that you must use for the next request if your captcha answer was incorrect. Although the captcha id returned should be the same as the one passed in the POST, you should always use the id given in the error response just in case the server changes the id on you. If you didn't pass an id in the POST, you'll be provided one in the error response.
	
	Example captcha error:
	
  ```json
	{
		"captcha": {
			"id": "abc123",
			"question": "If tomorrow is Saturday, what day is today?"
		},
		"reseller": {
			"name": "Greenview Data",
			"first_name": "Phil",
			"last_name": "Green",
			"email_address": "phil@vedit.com",
			"address": "8178 Jackson Rd",
			"city": "Ann Arbor",
			"state": "Michigan",
			"postal_code": "48103",
			"country": "United States",
			"phone": "734-426-7500",
			"username": "my-reseller-account",
			"application_name": "cpanel"
		},
		"errors": {
			"reseller": {},
			"captcha": {
			  "answer": [ "is not correct" ]
			}
		}
	}
  ```

  Example reseller validation error:
	
  ```json
	{
		"captcha": {
			"id": "abc123",
			"question": "If tomorrow is Saturday, what day is today?"
		},
		"reseller": {
			"name": "Greenview Data",
			"first_name": "Phil",
			"last_name": "Green",
			"email_address": "phil@vedit.com",
			"address": "8178 Jackson Rd",
			"city": "Ann Arbor",
			"state": "Michigan",
			"postal_code": "48103",
			"country": "United States",
			"phone": "734-426-7500",
			"username": "my-reseller-account",
			"application_name": "cpanel"
		},
		"errors": {
			"reseller": {
				"username": [ "must be specified" ]
			},
			"captcha": {}
		}
	}
  ```
	
	Required Fields:
	
    * name
    * email_address
    * username
