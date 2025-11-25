
Let’s say you have the following `main.tf` in directory `foo`:

```hcl
# foo/main.tf
resource "local_file" "file" {
  content  = "Hello, World!"
  filename = "${path.module}/hi.txt"
}
```

As we learned above, integrating this OpenTofu project with Terragrunt is as simple as creating a `terragrunt.hcl` file in the same directory:

```bash
touch foo/terragrunt.hcl
```

You can now run `terragrunt` commands within the `foo` directory, as if you were using `tofu` or `terraform`.

```bash
$ cd foo
$ terragrunt apply -auto-approve
```

**terraform oder tofu festlegen**

`export TG_TF_PATH=$(type -p terraform)`


https://terragrunt.gruntwork.io/docs/getting-started/quick-start/