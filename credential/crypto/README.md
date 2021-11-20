# git-credential-crypto

お使いのシステムで安全に利用できる [credential.helper](https://git-scm.com/docs/gitcredentials) が存在しない場合のみ、このスクリプトの導入を検討して下さい。


## Quickstart

```sh
# システムにログイン
ssh your-git-client

# Git credential.helper に設定する
sudo curl \
  https://raw.githubusercontent.com/oshinko/git-contrib/main/credential/crypto/git-credential-crypto \
  -o /usr/local/bin/git-credential-crypto
GIT_CREDENTIAL_HELPER=$_
sudo chmod +x $GIT_CREDENTIAL_HELPER
git config --global credential.helper $GIT_CREDENTIAL_HELPER

# 資格情報一覧の暗号化に使う秘密鍵 (パスワード) を入力する
read -sp "Enter encryption password: " GIT_CREDENTIAL_CRYPTO_PASSWORD; echo

# パスワード長を確認してエクスポート
if [ ${#GIT_CREDENTIAL_CRYPTO_PASSWORD} -lt 8 ]; then echo "暗号化パスワードが短すぎます。"; fi
export GIT_CREDENTIAL_CRYPTO_PASSWORD

# 資格情報一覧ファイルの場所を設定する
GIT_CREDENTIAL_CRYPTO_ENV_FILE=~/.git-credentials.env
cat << EOF > $GIT_CREDENTIAL_CRYPTO_ENV_FILE
export GIT_CREDENTIAL_CRYPTO_FILE=~/.git-credentials.enc
EOF
chmod 600 $GIT_CREDENTIAL_CRYPTO_ENV_FILE
. $GIT_CREDENTIAL_CRYPTO_ENV_FILE

# ログイン時に環境変数を読み込むようにする
PROFILE_FILE=~/.bash_profile
if ! cat $PROFILE_FILE | grep $GIT_CREDENTIAL_CRYPTO_ENV_FILE &>/dev/null; then
  cat << EOF >> $PROFILE_FILE

# For git-credential-crypto (Git credential.helper)
. $GIT_CREDENTIAL_CRYPTO_ENV_FILE
EOF
fi
```


## git-clone せずに資格情報を記憶する

```sh
# 資格情報を入力する
MY_GIT_PROTOCOL=https
MY_DEFAULT_GIT_HOSTNAME=github.com  # default
read -p "Enter your Git hostname [$MY_DEFAULT_GIT_HOSTNAME]: " MY_GIT_HOSTNAME
if [ "$MY_GIT_HOSTNAME" = "" ]; then
  MY_GIT_HOSTNAME=$MY_DEFAULT_GIT_HOSTNAME
fi
read -p "Enter $MY_GIT_HOSTNAME's username: " MY_GIT_USERNAME
read -sp "Enter $MY_GIT_HOSTNAME's password (or token): " MY_GIT_PASSWORD; echo

# 平文の資格情報を扱うため、履歴を残さないようにする
HISTIGNORE=*

# 資格情報を設定する
cat << EOF | eval $GIT_CREDENTIAL_HELPER store
protocol=$MY_GIT_PROTOCOL
host=$MY_GIT_HOSTNAME
username=$MY_GIT_USERNAME
password=$MY_GIT_PASSWORD
EOF

# 履歴を残すようにする
unset HISTIGNORE

# テスト
cat << EOF | git credential fill
protocol=$MY_GIT_PROTOCOL
host=$MY_GIT_HOSTNAME
EOF
```
