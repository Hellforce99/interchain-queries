# Task-9 Rehber
# 1-) binary indirelim 
         cd $HOME
         git clone https://github.com/Stride-Labs/interchain-queries.git
         cd interchain-queries
         go build
         cd
         sudo mv interchain-queries /usr/local/bin/icq
# 2-) icq anasayfası açalım
    cd $HOME && mkdir .icq
# 3-) yapılandırma dosyası yaratalım
    cd $HOME && mkdir .icq
    sudo tee $HOME/.icq/config.yaml > /dev/null <<EOF
    default_chain: stride-testnet
    chains:
      gaia-testnet:
        key: wallet
        chain-id: GAIA
        rpc-addr: http://127.0.0.1:buraya      # Gaia RPC yazacağız
        grpc-addr: http://127.0.0.1:buraya     # Gaia GRPC yazacağız
        account-prefix: cosmos
        keyring-backend: test
        gas-adjustment: 1.2
        gas-prices: 0.001uatom
        key-directory: /root/.icq/keys
        debug: false
        timeout: 20s
        block-timeout: ""
        output-format: json
        sign-mode: direct
      stride-testnet:
        key: wallet
        chain-id: STRIDE-TESTNET-4
        rpc-addr: http://127.0.0.1:buraya      # Stride RPC yazacağız
        grpc-addr: http://127.0.0.1:buraya     # Stride GRPC yazacağıze
        account-prefix: stride
        keyring-backend: test
        gas-adjustment: 1.2
        gas-prices: 0.001ustrd
        key-directory: /root/.icq/keys
        debug: false
        timeout: 20s
        block-timeout: ""
        output-format: json
        sign-mode: direct
    cl: {}
    EOF
# 4-) cüzdanı içe aktaralım
      icq keys restore --chain stride-testnet <your_wallet_name>
      icq keys restore --chain gaia-testnet <your_wallet_name>
# eğer "icq not found" hatası alırsanız
# mevcut icq kurulumunun tüm dosyalarını silin
      cd $HOME
      rm -rf interchain-queries
      rm -rf /usr/local/bin/icq
      rm -rf .icq
      rm -rf /etc/systemd/system/icqd.service
# go yükleyin
    wget -c https://go.dev/dl/go1.18.3.linux-amd64.tar.gz && rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.3.linux-amd64.tar.gz && rm -rf go1.18.3.linux-amd64.tar.gz
    echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
    echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
    echo 'export GO111MODULE=on' >> $HOME/.bash_profile
    echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
# ve icq'yi tekrar kurun ve cüzdanı içe aktarın
# 5-) icq dosyasını oluşturalım
      sudo tee /etc/systemd/system/icqd.service > /dev/null <<EOF
      [Unit]
      Description=Interchain Query Service
      After=network-online.target
      [Service]
      User=$USER
      ExecStart=$(which icq) run --debug
      Restart=on-failure
      RestartSec=3
      LimitNOFILE=65535
      [Install]
      WantedBy=multi-user.target
      EOF
# 6-) icq başlatalım
      sudo systemctl daemon-reload
      sudo systemctl enable icqd
      sudo systemctl restart icqd
# 7-) güncelleştirme
      rly transact update-clients stride-gaia
# 8-) logları kontrol edelim (20 dk sürebilir çıktıyı almak)
      journalctl -u icqd -f -o cat
# 9-) transfer yapabilirsiniz tx leri explorer kontrol edin
      rly transact transfer gaia stride 1000uatom <yourstrideaddress> channel-0 --path stride-gaia
      rly transact transfer stride gaia 1000ustrd <yourcosmosaddress> channel-0 --path stride-gaia
