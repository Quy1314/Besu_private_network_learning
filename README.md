# Besu_private_network_learning

## 📌 Installation (Ubuntu)

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install -y openjdk-17-jdk curl wget unzip git build-essential

# Download Hyperledger Besu (v25.9.0)
wget https://github.com/hyperledger/besu/releases/download/25.9.0/besu-25.9.0.tar.gz

# Extract and move to /usr/local
tar -xvf besu-25.9.0.tar.gz
sudo mv besu-25.9.0 /usr/local/besu

# Add Besu to PATH
echo 'export PATH=$PATH:/usr/local/besu/bin' >> ~/.bashrc
source ~/.bashrc

# Verify installation
besu --version
