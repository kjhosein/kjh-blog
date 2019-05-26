
---
title: "Terraform Tips and Tricks"
date: 2018-12-01T10:23:05-04:00
draft: false
---

> *Update, Feb 2019*: Note that with the imminent release of Terraform v0.12, some of the advice, commands, etc. in this article may not apply or work any more.

Having used Hashicorp's Terraform since 2016, I've collected a set of tips and tricks around developing and debugging Terraform code.


## Version Pinning
Even at 4+ yrs old, Terraform is still not even at version 1.0. However, along the way even the minor releases have had major--and many times breaking--changes. For that reason, I've always strongly recommended that one pins their code to at least the minor version # (e.g. 0.11.x), and Terraform has a mechanism to do this. E.g.:

```
terraform {
  required_version = ">= 0.11.7, < 0.12" # any version from 0.11.7 up to, but not including 0.12
}
```

You should also pin your providers' versions as well, although I've found you can (and should) be more lenient and allow for anything within a major version (e.g. 2.x). E.g.:
```
provider "azurerm" {
  version         = "~> 1.2"  # any non-beta version >= 1.2.0 and < 2.0.0
}
```

Additionally, you can also enforce a specific version of the TF binary to use in your code dir by using a  3rd-party tool called [tfenv](https://github.com/tfutils/tfenv). Don't procrastinate on this one as it only takes minutes to set up.


## Aliases
If you're a regular Terraform user, then you'll want to save yourself some keystrokes and create some shell aliases. Here are mine:

```
alias tf='terraform'
alias tfa='time terraform apply && say "Terraform has completed"' # The 'say' command may not be available on all *nix systems.
alias tfd='terraform-docs'  # 'terraform-docs' command generates documentation from your TF code
alias tfg='terraform graph -draw-cycles | dot -Tpng > graph.png; open graph.png'
alias tfi='terraform init --upgrade'
alias tfp='echo running terraform get update then plan; terraform get -update && terraform plan -out /tmp/zplan'
alias tfv='terraform validate'
```


## Did You Forget to ... ?

`terraform get --update`

If you updated your modules, then you may need to run this command to get the updated code added into the `.terraform` directory. Running `terraform get` alone will not pull in the updated code if the module already exists in the `.terraform` dir.  See the [TF get command's docs](https://www.terraform.io/docs/commands/get.html) for more info.


`terraform init --upgrade`

If you made changes to your working directory's configuration, then you'll need to include the `--upgrade` flag in order to pull in the latest versions of your plugins and modules. For example, you may want the latest version of a provider to be installed.

See the [TF init command's docs](https://www.terraform.io/docs/commands/init.html) for more info.


## Debugging

When you're stuck using typical dev & test methods, it's time to turn on Terraform debugging! This is fairly easy to do with these 2 commands:

```
export TF_LOG=DEBUG
export TF_LOG_PATH=/tmp/tf.debug.txt
```

This will log debug-level output to the `/tmp/tf.debug.txt` file, which you can point to a file and location of your choice.

When you're done, you can turn off debugging with these commands:
```
unset TF_LOG
```

Next, if `terraform apply` is returning a vague or seemingly nonsensical error, try creating the same cloud resources using the provider's CLI. You may be hitting some limitation with the provider and/or your account. Attempting to create the resources using the CLI may spit back more useful error messages. Sometimes the Go SDK (that Terraform uses to talk to the cloud providers) has bugs that prevent a useful error from being passed through.


## Developing Tricky Code

If your project is particularly involved and you're trying to add a complex piece of Terraform code, for example an interpolation, then [`terraform console`](https://www.terraform.io/docs/commands/console.html) can be your best friend.

Here's how you would set it up:

Switch to a sandbox directory and create one (1) or more `*.tf` files and/or a `terraform.tfvars` file with whatever variables you need. 

For example, you could have:

`test.tf`:
```
variable a {}
variable b { default = 5 }
```

`terraform.tfvars`:
```
a = "blah"
```

Now you can run `terraform console` _in that directory_:
```
$ terraform console
> var.a
blah
> var.b
5
> var.a * pow( 2, 4 )
80
> "${var.a * pow( 2, 4 )}"
80
```

## Other Useful Commands

`terraform validate` 

This is a quick way to ensure that your syntax is good and that all of your variables are declared. However, it won't detect where you have underlying issues or missing dependencies in your target environment.

`terraform graph`

At some point, your code will become more complex and you will have dependencies amongst your resources. In general, you'll want to have implicit dependencies such that Terraform builds resources in a certain order. 

For example, in AWS, if you're building an EC2 instance that is referencing a Security Group that you're also building in the same code, the EC2 instance must implicitly reference the SG or you may end up with an error saying that the SG does not exist. 

Viewing the graph of your code will verify whether or not you actually have that dependency.

`terraform-docs`

This is a 3rd-party tool that generates documentation (Markdown or JSON) from the comments and the variables in your code. You can _Brew_ this up on a Mac or snag it from [its GitHub repo](https://github.com/segmentio/terraform-docs).


## Resources

* [Terraform Interpolation documentation](https://www.terraform.io/docs/configuration-0-11/interpolation.html). # For any pre-v0.12 code, this is probably the page where I spend the most time. For v0.12, you'll want to check out the [Expressions docs](https://www.terraform.io/docs/configuration/expressions.html)
* The [Terraform user group/mailing list](https://groups.google.com/forum/#!forum/terraform-tool) is a great place to ask for help.
* If you're an IRCer, you can jump into the [Gitter channel](https://gitter.im/hashicorp-terraform/Lobby).
* [StackOverflow](https://stackoverflow.com/questions/tagged/terraform) is also quite helpful.
* If you're confident you've found an issue or want to request a new feature, head to the [GitHub issues list](https://github.com/hashicorp/terraform/issues/).
