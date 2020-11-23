# **ML Model Serving**

---

## TensorFlow Serving with Docker
```
# Variables
gcs_path_to_model="gs://model_artificats/bert_model_z1.tar.gz"
gcs_model_filename=${gcs_path_to_model##*/}
model_path="$(pwd)/saved_model/1/bert_model_z1"
model_name=bert_model

# Install Docker (if not already installed)
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install -y wget apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
# Add docker to sudo users
sudo groupadd docker
sudo usermod -aG docker $USER
# Test Install
#sudo docker run hello-world

# Pull Docker dependencies and run container for model serving
sudo docker pull tensorflow/serving
gsutil cp -r "$gcs_path_to_model" .
tar -zxvf $gcs_model_filename
# Mkdir with version number (1 = v1)
mkdir 1
mv "$model_path"/* 1
mv 1 "$model_path"
sudo docker run -t --rm -p 8501:8501 -v "$model_path:/models/$model_name" -e MODEL_NAME=$model_name tensorflow/serving &
```

# Tensorflow Client (installation and testing)
```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
chmod +x Miniconda3-latest-Linux-x86_64.sh
sudo ./Miniconda3-latest-Linux-x86_64.sh -b -p /opt/miniconda
sudo /opt/miniconda/bin/pip install tensorflow

curl -d '{"instances": [1.0]}' -X POST http://localhost:8501/v1/models/$model_name:predict
```
