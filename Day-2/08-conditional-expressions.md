# Conditional Expressions

Conditional expressions in Terraform are used to define conditional logic within your configurations. They allow you to make decisions or set values based on conditions. Conditional expressions are typically used to control whether resources are created or configured based on the evaluation of a condition.

The syntax for a conditional expression in Terraform is:

```hcl
condition ? true_val : false_val
```

- `condition` is an expression that evaluates to either `true` or `false`.
- `true_val` is the value that is returned if the condition is `true`.
- `false_val` is the value that is returned if the condition is `false`.

Here are some common use cases and examples of how to use conditional expressions in Terraform:

## Conditional Resource Creation Example

```hcl
resource "aws_instance" "example" {
  count = var.create_instance ? 1 : 0

  ami           = "ami-XXXXXXXXXXXXXXXXX"
  instance_type = "t2.micro"
}
```

In this example, the `count` attribute of the `aws_instance` resource uses a conditional expression. If the `create_instance` variable is `true`, it creates one EC2 instance. If `create_instance` is `false`, it creates zero instances, effectively skipping resource creation.

# Conditional Variable Assignment Example

```hcl
variable "environment" {
  description = "Environment type"
  type        = string
  default     = "development"
}

variable "production_subnet_cidr" {
  description = "CIDR block for production subnet"
  type        = string
  default     = "10.0.1.0/24"
}

variable "development_subnet_cidr" {
  description = "CIDR block for development subnet"
  type        = string
  default     = "10.0.2.0/24"
}

resource "aws_security_group" "example" {
  name        = "example-sg"
  description = "Example security group"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = var.environment == "production" ? [var.production_subnet_cidr] : [var.development_subnet_cidr]
  }
}

```

In this example, the `locals` block uses a conditional expression to assign a value to the `subnet_cidr` local variable based on the value of the `environment` variable. If `environment` is set to `"production"`, it uses the `production_subnet_cidr` variable; otherwise, it uses the `development_subnet_cidr` variable.

## Conditional Resource Configuration 

```hcl
resource "aws_security_group" "example" {
  name = "example-sg"
  description = "Example security group"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = var.enable_ssh ? ["0.0.0.0/0"] : []
  }
}
```

In this example, the `ingress` block within the `aws_security_group` resource uses a conditional expression to control whether SSH access is allowed. If `enable_ssh` is `true`, it allows SSH traffic from any source (`"0.0.0.0/0"`); otherwise, it allows no inbound traffic.

Conditional expressions in Terraform provide a powerful way to make decisions and customize your infrastructure deployments based on various conditions and variables. They enhance the flexibility and reusability of your Terraform configurations.

### Extra Note :-

## Let’s explore conditional expressions with a few real-world examples:

# 1. Selecting a Value Based on a Condition
One common use of a conditional expression is to select between two values based on a condition. For example, let's say we want to choose an instance type based on the environment (either production or development).

Example: Instance Type Based on Environment

'''hcl
variable "environment" {
  type    = string
  default = "development"
}

resource "aws_instance" "example" {
  ami           = "ami-0123456789abcdef0"
  instance_type = var.environment == "production" ? "m5.large" : "t2.micro"

  tags = {
    Name = "MyInstance"
  }
}
'''

Explanation:
If var.environment is "production", the instance type will be "m5.large".
Otherwise (if the environment is anything else, like "development"), the instance type will be "t2.micro".

# 2. Conditionally Set a Resource Attribute
You can use a conditional expression to dynamically set a resource attribute. For instance, you might want to conditionally assign a value to a resource attribute like a security group rule, depending on whether a feature is enabled.

Example: Security Group Rule Based on a Condition

'''hcl
variable "enable_https" {
  type    = bool
  default = true
}

resource "aws_security_group" "example" {
  name        = "example-security-group"
  description = "Allow web traffic"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  Conditionally allow HTTPS based on the variable
  ingress {
    from_port   = var.enable_https ? 443 : null  # If enabled, allow HTTPS, else skip
    to_port     = var.enable_https ? 443 : null
    protocol    = var.enable_https ? "tcp" : null
    cidr_blocks = var.enable_https ? ["0.0.0.0/0"] : []
  }
}
'''

Explanation:
If var.enable_https is true, the rule for port 443 (HTTPS) will be added.
If var.enable_https is false, the rule will be skipped (because null and an empty list [] are ignored).

# 3. Default Values Based on Condition
You might want to assign a default value to a variable or resource attribute based on a condition, for example, setting a default size for a disk unless it is explicitly provided.

Example: Default Disk Size

'''hcl
variable "disk_size" {
  type    = number
  default = 20
}

resource "aws_instance" "example" {
  ami           = "ami-0123456789abcdef0"
  instance_type = "t2.micro"
  
  block_device {
    device_name = "/dev/sdh"
    volume_size = var.disk_size != 0 ? var.disk_size : 30  # Default to 30 if disk_size is 0
  }
}
'''

Explanation:
If var.disk_size is provided and is not 0, the instance will have the size defined in the disk_size variable.
If var.disk_size is 0, the instance will be created with a default disk size of 30 GB.

# 4. Switching Between Multiple Values
You can use nested conditional expressions to switch between more than two values. For example, based on different environments, you might want to choose different AMIs.

Example: Selecting an AMI Based on the Environment

'''hcl
variable "environment" {
  type    = string
  default = "development"
}

resource "aws_instance" "example" {
  ami           = var.environment == "production" ? "ami-prod-12345" :
                  var.environment == "staging"   ? "ami-staging-12345" :
                  "ami-dev-12345"  # Default for development

  instance_type = "t2.micro"
}
'''

Explanation:
If var.environment is "production", the AMI will be "ami-prod-12345".
If var.environment is "staging", the AMI will be "ami-staging-12345".
If it's neither of these (i.e., "development"), the default AMI "ami-dev-12345" is used.

# 5. Creating Optional Resources
In some cases, you may want to create a resource conditionally, such as creating an additional EC2 instance or a security group only when a specific condition is met.

Example: Creating a Resource Conditionally

'''hcl
variable "create_extra_instance" {
  type    = bool
  default = false
}

resource "aws_instance" "primary" {
  ami           = "ami-0123456789abcdef0"
  instance_type = "t2.micro"
}

resource "aws_instance" "extra" {
  ami           = "ami-0123456789abcdef0"
  instance_type = "t2.micro"

  count = var.create_extra_instance ? 1 : 0  # Create the instance only if true

  depends_on = [aws_instance.primary]
}
'''

Explanation:
If var.create_extra_instance is true, an additional instance (aws_instance.extra) will be created (because count = 1).
If var.create_extra_instance is false, the count will be 0, so the extra instance will not be created.

# Summary of Conditional Expression Benefits
Dynamic Configuration: You can change the behavior or values of your resources depending on the environment, input variables, or other conditions.
Simplifying Code: Avoids having multiple resources with repetitive configurations by dynamically assigning values.
Improved Flexibility: Makes your Terraform configurations more adaptable to different use cases (like switching between environments or enabling/disabling features).
In Terraform, conditional expressions allow you to create flexible and reusable configurations that adapt based on conditions, making your infrastructure as code more dynamic and powerful.
