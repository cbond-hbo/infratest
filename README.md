# Infratest

Infratest is a Golang library that we hope makes testing your infrastructure using
tests that are written in Golang easier to do. The general goals of the project are to
enable testing across technologies such as:

* AWS, including (but not limited to):
  * EC2
  * IAM
  * RDS
  * Route53
  * S3
  * DynamoDB
* Databases such as:
  * Mongodb
  * PostgreSQL
  * Cassandra

The library takes the approach of enabling simple assertion-based testing; that
is, asserting that some attribute of a resource is an expected value, that such
an attribute even exists, or that a given system responds to a request in a way
that you expect. Examples of things we might want to include are:

* Asserting that a resource name matches a pattern or set of rules.
* Asserting that an AWS IAM Policy contains a given combination of an Action,
  Resource, and Effect.
* Asserting that one can connect to a database endpoint and perform some
  arbitrary action using provided credentials.

> **Note:** While many of the resources that the library enables testing of
> can be managed by tools such as [Terraform](https://terraform.io) or
> [Ansible](https://ansible.com), the project explicitly does not include
> libraries for interacting with or executing deployments with Terraform or any
> other Infrastructure-As-Code tools. Instead, we recommend other libraries are
> used to perform that function, such as the excellent
> [Terratest](https://terratest.gruntwork.io) framework.

## Contribution Guidelines

We welcome pull requests for new features! Please review the following
guidelines carefully to ensure the fastest acceptance of your code.

### Package naming

In addition to following the general [Effective
Go](https://golang.org/doc/effective_go#package-names) guidelines for package
naming (i.e. all lowercase and no punctuation) package names must also indicate
the specific vendor or technology that the contained methods test. For example, 
all our code that tests AWS-based resources must be under the `aws` package.

### Function Naming

All publicly consumable functions must follow the naming pattern
`Assert[Entity][Action statement]`, where `[Entity]` is the object the assertion
is run against, and `[Action statement]` is a concise, descriptive name for what
the assertion tests.

Examples of good function names:

- AssertIAMPolicyContainsResourceAction
- AssertEC2TagValue
- AssertUserCanConnect
- AssertS3BucketPublicAccessBlocked

Note that these names do not include specific product or company names, e.g.
`AssertAWSIAMPolicyContainsResourceAction`, or `AssertPostgreSQLUserCanConnect`.
**This is intentional**. Because we separate different technologies or vendors
into discrete packages, the use of the vendor or technology name in the function
name is redundant and makes the name longer.

*For internal functions only*, any method which returns an error object must
have the letter `E` as the last character of its name.

### Function signatures

- Public functions must have `t *testing.T` as the first input parameter.
- Any function which interacts with outside resources must have a context object
  `ctx *context.Context` as an input, which should be directly after the test
  object for public functions.
- Any function which requires a third-party client, such as an AWS SDK client or
  database client, must accept this client object directly after the context
  object (which would be required since this by definition interacts with
  outside resources). This client object must be an interface type as discussed
  in the section on [the use of interfaces over direct
  clients](#use-of-interaces-rather-than-direct-types).

### Common library usage

To keep the number of dependencies low, the following standard libraries must be
used unless there is a compelling reason to use something else.

| Library Name  / URL                             | Used For                                                                  |
|-------------------------------------------------|---------------------------------------------------------------------------|
| github.com/stretchr/testify                     | Asserting actual values equal some expected value.                        |
| github.com/aws/aws-sdk-go-v2                    | All AWS related interactions.                                             |
| gopkg.in/square/go-jose.v2/json                 | JSON manipulation, marshalling, etc                                       |
| k8s.io/client-go                                | Kubernetes interactions                                                   |

### Use of interfaces rather than direct types

Where we interact with outside libraries that themselves interact with external
resources, we require use of a limited interface type. Referencing the third party 
type directly. (An example of this would be the AWS SDK.) 
We do this so that we can easily unit test our methods without relying on provisioning 
external resources.

### Tests

All methods, especially ones that are public-facing, must have associated unit
tests. These tests must not rely on the existence of external resources unless
absolutely required; by using interfaces as described in the [previous
section](#use-of-interaces-rather-than-direct-types) this should be simple to
accomplish.


### Contributing

Contributions are welcome! Please follow this process:

* Fork the repository into your own user.
* Create a new branch off of `main` in your repository; **we do not accept contributions off of forks' `main` branches** (it also makes merging future changes from our repository to yours more difficult).
* Make your changes and commit them, including updates to any tests.
* Run the testing suite on at least any of the packages that you updated or added.
* Open a pull request back to our repository. Please reference any issue that you are fixing.
