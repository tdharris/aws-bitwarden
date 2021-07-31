# AWS Bitwarden Profile
Script to source aws credential profiles from bitwarden using [bw cli](https://github.com/bitwarden/cli).

See AWS Documentation [Sourcing credentials with an external process](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sourcing-external.html) for more details on the approach.

## Installation
- Download [aws-bw](https://raw.githubusercontent.com/tdharris/aws-bitwarden/master/aws-bw)
- Make it executable and reachable. E.g. `chmod +x aws-bw && sudo cp aws-bw /usr/local/bin`
- Download and install the Bitwarden CLI [bw](https://github.com/bitwarden/cli#downloadinstall) and [jq](https://stedolan.github.io/jq/download/) for parsing JSON.

## Usage

### Adding AWS Secrets to Bitwarden Vault
![](/assets/bw-item-ss.png "bitwarden item for aws-bw")
- Create an item to store a single aws credential profile. Make the name unique so it is easy to find. E.g. `aws-iam-personal-mycli-env` or a naming convention like `<clientA>-aws-iam-<account>-<name>-env` perhaps.
- Add the following custom fields and provide their values: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`

### Getting secrets onto server
#### Initial Configuration
- If the preferred approach is to set Environment Variables, then consider [envwarden](https://github.com/envwarden/envwarden).
- If the preferred approach is with the `.aws/credentials` file, then configure it to use the `aws-bw` script and pass the relevant Bitwarden Item Name to lookup. E.g.
```
[default]
credential_process = aws-bw -s 'aws-iam-personal-mycli-env'
[profileA]
credential_process = aws-bw -s '<bw-item-name>'
```
*Note: The Bitwarden Item Name is expected to have both `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` as Custom Fields.*
#### General Use-Case Steps
- First unlock the vault to generate a session key with Bitwarden & set the environment variable:
```
export BW_SESSION="$(bw unlock --raw)"
```
*Note: Unfortunately AWS doesn't appear to pass-along any prompts that an external script might need from the user so this must be done as a manual step prior to using the script.*

- Now you can use the AWS CLI and specify the profile:

```
aws sts get-caller-identity --profile profileA
```
- When finished, to destroy any active Bitwarden session keys:
```
bw lock
unset BW_SESSION
```
