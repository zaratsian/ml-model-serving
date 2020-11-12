# **ML Model Serving**

---

## TensorFlow Serving with Docker
```
github_repo=https://github.com/zaratsian/ml-model-serving.git
gcs_path_to_model="gs://model_artificats/bert_modelv1.0"
model_path="$(pwd)/bert_modelv1.0"
model_name=bert_model

# Clone model repo
sudo apt install git -y
git clone "$github_repo"

# Install Docker (if not already installed)
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
# Add docker to sudo users
sudo groupadd docker
sudo usermod -aG docker $USER
# Test Install
#sudo docker run hello-world

# Pull Docker dependencies and run container for model serving
docker pull tensorflow/serving
gsutil cp -r "$gcs_path_to_model" .
# Mkdir with version number (1 = v1)
mkdir 1
mv "$model_path"/* 1
mv 1 "$model_path"
docker run -t --rm -p 8501:8501 -v "$model_path:/models/$model_name" -e MODEL_NAME=$model_name tensorflow/serving &
```

# Tensorflow Test Model Endpoint
```
curl -d '{"instances": [1.0]}' -X POST http://localhost:8501/v1/models/$model_name:predict
```
