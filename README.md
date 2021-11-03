## Auditoria de senhas do Active Directory


Instruções de como avaliar a qualidade das senhas dos colaboradores da empresa e contas de serviço.

#### Gerar o DUMP da Base de Dados do Active Directory

Executar os comandos via prompt dentro do Active Directory:

```
ntdsutil "ac i ntds" "ifm" "create full c:\TEMP_DUMP_AD" q q
reg save HKLM\SAM c:\TEMP_DUMP_AD\SAM
reg save HKLM\SYSTEM c:\TEMP_DUMP_AD\SYSTEM
reg save HKLM\SECURITY c:\TEMP_DUMP_AD\SECURITY
```

O tratamento dos dados deve ser feito utilizando a ferramenta "Impacket".

Projeto Github: https://github.com/SecureAuthCorp/impacket

Gerando arquivo de OUTPUT para ser utilizado no HashCat para as tentativas de quebras de senhas:

```
python3 secretsdump.py -system /TEMP_DUMP_AD/registry/SYSTEM -ntds /TEMP_DUMP_AD/Active\ Directory/ntds.dit LOCAL -outputfile HASH-OUTPUT-FILE
```

Utilizando agora o HashCat para quebrar as senhas:

```
hashcat -a 0 -m 1000 --status --force -o CRACKED-PASSWORDS-RESULT.txt HASH-OUTPUT-FILE.ntds wordlist/senhas-comuns.txt
```

-o CRACKED-PASSWORDS-RESULT.txt **->** Será o resultados as senhas que o HashCat conseguiu quebrar.
wordlist/senhas-comuns.txt **->** É a wordlist que usamos com palavras e senhas comuns para o ambiente da empresa. "wordlist/senhas-comuns.txt"
HASH-OUTPUT-FILE.ntds **->** É o arquivo gerado pelo Impacket, a matriz que vamos utilizar de base para o HashCat.

#### Instalação do Hashcat e Impacket:

```
#!/bin/bash

# Instalação do Hashcat em um servidor Gnu/Linux com alto processamento gráfico (GPU)
# Exemplo na Azure: Standard_E8ds_v4

sudo apt update
sudo apt install hashcat hashcat-nvidia -y

# Instalação do Impacket via PIP ou Pacote
# sudo apt install python3-pip -y
# sudo pip3 install impacket

# wget https://github.com/SecureAuthCorp/impacket/releases/download/impacket_0_9_22/impacket-0.9.22.tar.gz
# tar zxvf impacket-0.9.22.tar.gz
# Instalação do Impacket via PIP ou Pacote

# Site com diversas wordlists genéricas
# https://weakpass.com/lists
# wget https://download.weakpass.com/wordlists/1315/only_latin.Top.gz
# gunzip only_latin.Top.gz
# Site com diversas wordlists genéricas

# comandos do hashcat
# Hashcat NLTM Hashs:
# hashcat -a 0 -m 1000 --status --force -o CRACKED-PASSWORDS-RESULT.txt HASH-OUTPUT-FILE.ntds senhas-comuns.txt
```

#### Script para tratamento dos resultados

Script de apoio para ajudar no tratamento dos resultados após o output do Hashcat.

```
#!/bin/bash

hash_found="/home/kali/Desktop/TEMP_DUMP_AD/dump/CRACKED-PASSWORDS-RESULT.txt" # Founds do Hashcat
ntds_output_file="/home/kali/Desktop/TEMP_DUMP_AD/dump/hashs-output.ntds" # Arquivo matriz do Dump do Active Directory

echo "[+] RedTeam - 2021 | Active Directory Password Audit [+]"
echo ""

while IFS=":" read pass_hash pass_plaintext
do
        echo "[+] ------------------------------------------------------------------------------------------ [+]"
        echo "[+] PassHash: $pass_hash | PassHash PlainText: $pass_plaintext"
        echo ""
        validation=$(cat $ntds_output_file | egrep $pass_hash | cut -d ":" -f1 | cut -d "\\" -f2)
        printf '%s\n' $validation
        
        #domain_name=$(cat $ntds_output_file | egrep $pass_hash | cut -d ":" -f1 | cut -d "\\" -f1)
        #username=$(cat $ntds_output_file | egrep $pass_hash | cut -d ":" -f1 | cut -d "\\" -f2)
        #printf '%s\n' $domain_name $username        
        
        #echo "[+] ------------------------------------------------------------------------------------------ [+]"
done < $hash_found
echo ""
echo "[+] ------------------------------------------------------------------------------------------ [+]"
```
