# 📱 Instalação de Custom ROM em Samsung via Linux (Heimdall + ADB)

Este guia documenta o processo completo para substituir o sistema operacional de fábrica de um **aparelho Samsung Galaxy** por uma **Custom ROM**, usando ferramentas nativas do ecossistema Linux: **Heimdall** e **ADB**.

A ideia geral é sempre a mesma, em qualquer modelo:

1. Instalar um **custom recovery** (ex.: TWRP) com o Heimdall.
2. **Limpar** o sistema antigo pelo recovery.
3. **Enviar** a ROM (e os GApps, se precisar) com o ADB.
4. **Instalar** os arquivos pelo recovery e reiniciar.

![Plataforma](https://img.shields.io/badge/plataforma-Linux-blue)
![Ferramentas](https://img.shields.io/badge/ferramentas-Heimdall%20%7C%20ADB%20%7C%20TWRP-orange)

> 💡 No final há um [exemplo prático](#-exemplo-prático-galaxy-tab-e-sm-t560--dotos) com um caso real (Galaxy Tab E SM-T560 + DotOS).

---

## ⚠️ Aviso importante

Este procedimento **apaga todos os dados** do aparelho e pode **anular a garantia**. Existe risco de deixar o dispositivo inutilizável (*brick*). Faça backup do que for importante e prossiga por sua conta e risco.

**Regra de ouro:** use apenas arquivos (ROM, recovery, GApps) feitos **para o modelo e a variante exatos** do seu aparelho. Arquivos de outro modelo, mesmo parecido, podem causar brick.

---

## 📋 Índice

1. [Glossário rápido](#1-glossário-rápido)
2. [Pré-requisitos e preparação do dispositivo](#2-pré-requisitos-e-preparação-do-dispositivo)
3. [Instalação do custom recovery via Heimdall](#3-instalação-do-custom-recovery-via-heimdall)
4. [Limpeza e transferência do sistema no recovery](#4-limpeza-e-transferência-do-sistema-no-recovery)
5. [Instalação da ROM e finalização](#5-instalação-da-rom-e-finalização)
6. [Solução de problemas](#6-solução-de-problemas)
7. [Sobre os arquivos `.zip` deste repositório (Git LFS)](#7-sobre-os-arquivos-zip-deste-repositório-git-lfs)
8. [Resumo dos comandos](#-resumo-dos-comandos)
9. [Exemplo prático](#-exemplo-prático-galaxy-tab-e-sm-t560--dotos)

---

## 1. Glossário rápido

| Termo | O que é |
|-------|---------|
| **ROM / Custom ROM** | Sistema operacional alternativo que substitui o de fábrica (ex.: LineageOS, DotOS) |
| **Heimdall** | Programa de Linux que grava arquivos no aparelho Samsung em Modo Download (equivalente ao Odin do Windows) |
| **ADB** | Ferramenta que conversa com o aparelho pelo USB (enviar arquivos, listar dispositivos etc.) |
| **Modo Download** | Tela especial do Samsung onde o PC consegue gravar partições no aparelho |
| **Recovery (TWRP)** | Mini sistema usado para limpar partições e instalar arquivos `.zip` |
| **GApps** | Pacote com Play Store e serviços do Google, necessário quando a ROM não os inclui |
| **Wipe** | Limpeza das partições do sistema antigo antes de instalar a ROM |
| **FRP** | Bloqueio de segurança que pede a conta Google antiga depois de resetar o aparelho |

---

## 2. Pré-requisitos e preparação do dispositivo

### 2.1 Compatibilidade

- O **Heimdall** funciona com aparelhos **Samsung** em Modo Download. Para outras marcas, o processo costuma usar `fastboot`, e este guia não se aplica diretamente.
- Aparelhos Samsung mais novos podem ter proteções extras (bloqueio de bootloader, Knox, partições dinâmicas) e exigir etapas adicionais. Consulte sempre a página da ROM e do recovery para o seu modelo.

### 2.2 Configurações do aparelho

**[Hardware / Android]**

1. Acesse **Configurações > Sobre o dispositivo** e toque **7 vezes** em **Número da versão** para ativar as **Opções do desenvolvedor**.
2. Em **Opções do desenvolvedor**, ative a **Depuração USB**. *(Permite que o PC converse com o aparelho via ADB.)*
3. Se existir a opção **Desbloqueio OEM**, ative-a.
4. Acesse **Configurações > Contas** e **remova a Conta Google** (e a conta Samsung, se houver). *(Evita o bloqueio FRP depois da instalação.)*
5. Faça backup e deixe a bateria com carga suficiente (de preferência acima de 50%).

### 2.3 Download dos arquivos requeridos

| Item | Descrição |
|------|-----------|
| **ROM customizada** | `.zip` da ROM, compatível com o **modelo exato** do hardware |
| **Custom recovery** | Ex.: TWRP, em `.tar` ou `.img`, específico para o modelo |
| **GApps** *(opcional)* | Pacote minimalista com os serviços do Google. Deve bater com a **arquitetura** (ARM, ARM64), a **versão do Android** e a **variante** (ex.: `pico`) indicadas pela ROM |

> Se a página de download informar o **checksum** (SHA256/MD5), confira o arquivo baixado.

### 2.4 Instalação das ferramentas no Linux

**[Terminal Linux]**

```bash
sudo apt update
sudo apt install heimdall-flash google-android-platform-tools-installer
```

> Em outras distribuições, instale os pacotes equivalentes (ex.: `heimdall` e `android-tools`).

✅ **Verificação:** execute `heimdall version` e `adb version`. Ambos devem mostrar a versão instalada.

---

## 3. Instalação do custom recovery via Heimdall

### 3.1 Extrair o arquivo do recovery

**[Preparo do arquivo .img]**

O Heimdall precisa do arquivo `.img`. Se o recovery veio em `.tar`, extraia:

```bash
tar -xvf <arquivo_do_recovery>.tar
```

✅ **Verificação:** o arquivo `recovery.img` (ou nome parecido) está no diretório atual.

### 3.2 Entrar no Modo Download

**[Hardware]**

1. Desligue o aparelho por completo.
2. Pressione e mantenha a **combinação de teclas do seu modelo**. Exemplos comuns:
   - Com botão Home: **Volume Menor (-) + Home + Power**
   - Sem botão Home: **Volume Menor + Volume Maior** com o cabo USB conectado, ou **Volume Menor + Bixby + Power**
3. Na tela de aviso (*Warning*), pressione **Volume Maior (+)** para confirmar.

✅ **Verificação:** a tela mostra a mensagem **"Downloading..."**.

> A combinação varia por modelo. Se não funcionar, pesquise por "*<seu modelo> download mode*".

### 3.3 Conectar ao PC e verificar a detecção

**[Terminal Linux]**

Conecte o aparelho ao computador via cabo USB e verifique se o Heimdall o reconhece:

```bash
sudo heimdall detect
```

✅ **Verificação:** o terminal retorna `Device detected`.

### 3.4 (Opcional) Confirmar o nome da partição de recovery

**[Comando Heimdall]**

O nome da partição pode variar entre modelos. Para listar as partições do aparelho:

```bash
sudo heimdall print-pit
```

Procure a partição de recovery (normalmente `RECOVERY`).

### 3.5 Gravar o recovery

**[Comando Heimdall]**

```bash
sudo heimdall flash --RECOVERY recovery.img --no-reboot
```

| Parâmetro | Função |
|-----------|--------|
| `--RECOVERY` | Define a partição de destino no dispositivo (ajuste conforme o PIT) |
| `recovery.img` | Arquivo do recovery que será gravado |
| `--no-reboot` | Evita que o aparelho reinicie sozinho, o que faria o sistema original sobrescrever o recovery recém-instalado |

✅ **Verificação:** a sessão encerra com `RECOVERY upload successful` e `Ending session...`.

### 3.6 Acessar o recovery pela primeira vez

**[Hardware]**

Com o `--no-reboot`, o aparelho continua na tela de Download.

1. Desconecte o cabo USB.
2. Force o desligamento (em geral, segurando **Volume Menor (-) + Power**).
3. Assim que a tela apagar, **mude imediatamente** os botões para a combinação do recovery do seu modelo. Exemplo comum: **Volume Maior (+) + Home + Power**.
4. Solte os botões assim que o logotipo do recovery aparecer.

✅ **Verificação:** a interface gráfica do recovery (ex.: TWRP) é carregada na tela.

> Se o aparelho iniciar o Android de fábrica em vez do recovery, o recovery foi sobrescrito: repita os passos 3.2 a 3.5.

---

## 4. Limpeza e transferência do sistema no recovery

### 4.1 Limpeza do sistema anterior (Wipe)

**[Recovery]**

1. Na tela principal, altere o idioma se desejar (ex.: Português) e deslize para permitir modificações no sistema.
2. Acesse **Limpar (Wipe) > Limpeza Avançada (Advanced Wipe)**.
3. Marque as partições:
   - Dalvik / ART Cache
   - Sistema (System)
   - Dados (Data)
   - Cache
   - *(Opcional)* Memória Interna (Internal Storage)
4. Deslize a barra inferior para confirmar a limpeza.

> Algumas ROMs pedem passos extras (como *format data*). Siga a documentação da ROM escolhida.

### 4.2 Transferência de arquivos via ADB Push

**[Terminal Linux]**

Com o aparelho ainda no recovery e conectado ao computador via USB, verifique a detecção:

```bash
adb devices
```

Retorno esperado (o número de série será diferente):

```text
List of devices attached
XXXXXXXXXXXXXXXX    recovery
```

Envie a ROM e os GApps para a memória interna (`/sdcard/`):

```bash
adb push <arquivo_da_rom>.zip /sdcard/
adb push <arquivo_dos_gapps>.zip /sdcard/
```

✅ **Verificação:** o terminal indica **100%** de progresso ao concluir cada transferência.

---

## 5. Instalação da ROM e finalização

**[Recovery]**

1. No menu principal, toque em **Instalar (Install)**.
2. Navegue até `/sdcard/` e selecione o `.zip` da **ROM**.
3. Toque em **Adicionar mais zips (Add more Zips)** e selecione o `.zip` dos **GApps** (se necessário). *A ordem importa: ROM primeiro, GApps depois.*
4. Deslize para confirmar a instalação em fila (*Swipe to confirm Flash*).
5. Aguarde a conclusão da gravação do sistema e do pacote Google.
6. Ao finalizar, toque em **Reiniciar Sistema (Reboot System)**.

> 🕐 **Nota:** o primeiro boot pode levar de **5 a 10 minutos**. Depois, o aparelho carrega a tela de boas-vindas da ROM para a configuração inicial.

---

## 6. Solução de problemas

| Problema | Possível solução |
|----------|------------------|
| `heimdall detect` não encontra o aparelho | Troque o cabo/porta USB (prefira cabo original), use `sudo` e confirme que o aparelho está em Modo Download |
| Erro ao gravar o recovery | Confirme o nome da partição com `heimdall print-pit` e se o arquivo é do modelo certo |
| O recovery some e volta o sistema original | Faltou o `--no-reboot` ou a troca imediata de botões para entrar no recovery |
| `adb devices` não lista o aparelho | Troque cabo/porta, rode `adb kill-server && adb start-server` e confirme que o recovery está carregado |
| Erro ao instalar o `.zip` | Verifique se ROM e GApps são do modelo e da versão do Android corretos |
| Travou no logotipo | Aguarde alguns minutos; se persistir, volte ao recovery, refaça o wipe e reinstale |
| Pede conta Google após o boot (FRP) | Remova as contas antes do procedimento (passo 2.2) |

---

## 7. Sobre os arquivos `.zip` deste repositório (Git LFS)

As ROMs e os pacotes têm centenas de MB. O Git comum não lida bem com arquivos desse tamanho (e o GitHub bloqueia arquivos acima de 100 MiB). Por isso, os `.zip` deste repositório ficam no **Git LFS** (*Large File Storage*).

### Como funciona

- No repositório fica só um **ponteiro**: um arquivo de texto minúsculo que aponta para o arquivo real.
- O arquivo real fica num armazenamento separado e é baixado quando você clona ou faz `git lfs pull`.
- Quem decide quais arquivos viram ponteiro é o arquivo `.gitattributes`.

### Para baixar os arquivos (quem for usar o guia)

```bash
# 1) Instalar o Git LFS (Debian/Ubuntu)
sudo apt install git-lfs
git lfs install

# 2) Clonar o repositório
git clone https://github.com/<usuario>/<repositorio>.git
cd <repositorio>

# 3) Se os .zip vierem pequenos (só ponteiros), baixe os arquivos reais
git lfs pull
```

✅ **Verificação:** `ls -lh *.zip` deve mostrar arquivos de centenas de MB. Se mostrar poucos bytes, são só ponteiros: rode `git lfs pull`.

> ⚠️ O botão **Download ZIP** do GitHub **não inclui** os arquivos do LFS (traz só os ponteiros). Para baixar pelo navegador, abra o arquivo no GitHub e use o botão de download dele.

### Para adicionar novos arquivos (manutenção)

```bash
git lfs install                     # uma vez por máquina
git lfs track "*.zip"               # registra o padrão no .gitattributes
git add .gitattributes
git add <arquivo>.zip
git commit -m "Adiciona <arquivo> via LFS"
git push
```

Comandos úteis:

| Comando | Para que serve |
|---------|----------------|
| `git lfs ls-files --size` | Lista os arquivos rastreados pelo LFS e o tamanho de cada um |
| `git lfs track` | Mostra quais padrões estão rastreados |
| `git lfs migrate import --include="*.zip"` | Converte `.zip` que já foram commitados do jeito comum (reescreve o histórico) |

### Limites do GitHub

- O LFS tem **cota gratuita limitada** de armazenamento e de banda de download por mês. Confira os valores atuais em **Settings > Billing** da sua conta.
- Cada download de arquivo LFS conta na **cota do dono do repositório**. Se a cota acabar e não houver forma de pagamento cadastrada, os downloads do LFS ficam bloqueados até o mês seguinte.
- Alternativa: publicar os `.zip` em **GitHub Releases** e deixar só os links no README, o que poupa a cota do LFS.

---

## 🧰 Resumo dos comandos

```bash
# Ferramentas
sudo apt update
sudo apt install heimdall-flash google-android-platform-tools-installer

# Recovery (aparelho em Modo Download)
tar -xvf <recovery>.tar
sudo heimdall detect
sudo heimdall print-pit          # opcional: confirmar nome da partição
sudo heimdall flash --RECOVERY recovery.img --no-reboot

# Transferência (aparelho no recovery)
adb devices
adb push <rom>.zip /sdcard/
adb push <gapps>.zip /sdcard/
```

---

## 🧪 Exemplo prático: Galaxy Tab E (SM-T560) + DotOS

Este guia foi originalmente validado com:

| Item | Valor |
|------|-------|
| Aparelho | Samsung Galaxy Tab E (SM-T560) |
| Recovery | TWRP 3.0.2-1 (`twrp_3.0.2-1_sm-t560.tar`) |
| ROM | DotOS (`T560_DOTOS_v4.zip`) |
| GApps | Open GApps ARM, Android 7.1, variante pico (`open_gapps-arm-7.1-pico-20181025.zip`) |
| Modo Download | **Volume Menor (-) + Home + Power**, depois **Volume Maior (+)** para confirmar |
| Modo Recovery | **Volume Maior (+) + Home + Power** |

Retorno do `adb devices` nesse caso:

```text
List of devices attached
38019b6a4010a200    recovery
```

Comandos usados:

```bash
tar -xvf twrp_3.0.2-1_sm-t560.tar
sudo heimdall flash --RECOVERY recovery.img --no-reboot
adb push T560_DOTOS_v4.zip /sdcard/
adb push open_gapps-arm-7.1-pico-20181025.zip /sdcard/
```

---

## 📄 Licença

Este guia é distribuído para fins informativos. Os arquivos de ROM, GApps e recovery pertencem aos seus respectivos autores e seguem suas próprias licenças.

## 🤝 Contribuições

Testou em outro modelo ou ROM? Sugestões e correções são bem-vindas! Abra uma *issue* ou envie um *pull request*.
