# Fake AWS locally with LocalStack

### WSL
For this I used a WSL2 Ubuntu environment on a Windows 11 machine.

## Initial Setup
Docker, Git and Visual Studio Code need to be installed.

- #### Docker
https://docs.docker.com/desktop/windows/wsl/
Need to enable Kubernets, that will provide us with a single node cluster.
- #### Helm
https://github.com/helm/helm#install
- #### Git on WSL
```
sudo apt install git
git config --global user.name "Your Name"
git config --global user.email "youremail@domain.com"
```

It is recommended to install the latest Git for Windows in order to share credentials & settings between WSL and the Windows host. Git Credential Manager is included with Git for Windows and the latest version is included in each new Git for Windows release. During the installation, you will be asked to select a credential helper, with GCM set as the default. 
https://learn.microsoft.com/en-us/windows/wsl/tutorials/wsl-git 
- #### Visual Studio Code on WSL
https://code.visualstudio.com/docs/remote/wsl

- #### AWS CLI
https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html#cliv2-linux-install

Once the AWS CLI is installed, run aws configure to create some credentials.
https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html

## LocalStack Helm Charts
https://github.com/localstack/helm-charts

### Add Repo
```
helm repo add localstack https://localstack.github.io/helm-charts
helm repo update
```

If you need to specify some custom values for yor application you can pull the Chart:
```
helm pull localstack/localstack --untar
```

Here the vanilla Chart will be used
```
helm upgrade --install localstack localstack/localstack --namespace localstack --create-namespace
```

Run the output provided in order to get the application URL:
```
export NODE_PORT=$(kubectl get --namespace "localstack" -o jsonpath="{.spec.ports[0].nodePort}" services localstack)
export NODE_IP=127.0.0.1 #we are running locally
echo http://$NODE_IP:$NODE_PORT
```
We can save the URL in a environment variable in case we need it later
```
export LOCALSTACK_URL=http://$NODE_IP:$NODE_PORT
```

Associate the AWS CLI with my localstack running locally:
```
alias aws="aws --endpoint-url $LOCALSTACK_URL"
```

Some Buckets tests and commands:
```
# Check buckets (should be empty)
aws s3api list-buckets

# Create a Bucket
aws s3api create-bucket --bucket dot

#Create a file
echo "file to upload to my s3 bucket" >> test.txt

#Upload to Bucket
aws s3api put-object --bucket dot --key test --body test.txt

#Check objects on bucket
aws s3api list-objects --bucket dot

#Download the object with a new name
aws s3api get-object --bucket dot --key test test_from_s3.txt

#Delete the file on bucket
aws s3api delete-object --bucket dot --key test

#Delete Bucket
aws s3api delete-bucket --bucket dot
```