# 🌤️ Chatbot de Clima no Telegram com n8n

## 📝 Descrição do Projeto
Este projeto consiste em um chatbot construído no **n8n** e integrado ao **Telegram**, capaz de informar a temperatura atual de qualquer cidade do Brasil. 

O fluxo recebe a mensagem do usuário, realiza um tratamento de dados (removendo espaços, acentos e formatando a string), e consulta a API gratuita da **OpenWeather**. O diferencial deste workflow é a integração opcional com a inteligência artificial do **Google Gemini** para reescrever a resposta de forma mais natural e humanizada. O projeto também conta com tratamento de erros rigoroso e uma rota de *Fallback* determinística para garantir o funcionamento mesmo sem credenciais de IA.

---

## ⚙️ Pré-requisitos
Para rodar este projeto, você precisará de:
* Uma instância do [n8n](https://n8n.io/) rodando (local via Docker ou Cloud).
* Um Token de Bot do Telegram (gerado via `@BotFather`).
* Uma API Key da OpenWeather.

---

## 📥 Como Importar o Workflow
1. Faça o download do arquivo `workflow-chatbot-telegram.json` presente neste repositório.
2. Abra a sua interface do n8n.
3. No menu lateral esquerdo, clique em **Workflows** e depois em **Add Workflow**.
4. No canto superior direito, clique no menu de opções `(...)` e selecione **Import from File**.
5. Selecione o arquivo JSON baixado. O fluxo completo aparecerá na sua tela.

---

## 🔐 Configuração de Credenciais
Para garantir a segurança, nenhuma chave está exposta no código. O avaliador/usuário deverá configurar as credenciais nativamente no n8n.

### 1. Telegram Bot (`TELEGRAM_BOT_TOKEN`)
* Acesse os nós `Telegram Trigger` e os nós de envio de mensagem (`Send a text message`).
* Em **Credential for Telegram API**, crie uma nova credencial e insira o seu token fornecido pelo BotFather.

### 2. OpenWeather API (`OPENWEATHER_API_KEY`)
* Acesse o nó **HTTP Request**.
* Na lista de *Query Parameters*, localize a chave `appid`.
* Insira a sua API Key da OpenWeather no campo *Value* correspondente.

---

## 🧠 Uso do Google Gemini e Sistema de Fallback
Para enriquecer a experiência do usuário, este workflow utiliza um modelo LLM (**Google Gemini 2.5 Flash**) para formatar a mensagem final.
* **Onde está localizado:** O nó `Google Gemini Chat Model` está conectado à rota de sucesso do `HTTP Request`.
* **Como ativar:** Caso queira usar a IA, basta adicionar a sua credencial do Google AI Studio no nó do Gemini. 

**🛡️ Sistema de Fallback (Avaliação Segura):**
O workflow foi desenhado com uma arquitetura tolerante a falhas. Logo antes do nó da IA, existe um nó chamado `Code in JavaScript`. Ele gera a mensagem de temperatura de forma determinística e estática. 
Caso o nó do Gemini falhe (por exemplo, devido à ausência de credenciais por parte do avaliador), um nó condicional (`If1`) identifica a falha e redireciona o fluxo automaticamente para enviar a mensagem determinística. **Isso garante que o projeto possa ser avaliado sem custos adicionais e sem quebrar a automação.**

---

## 🚀 Como Executar e Testar o Chatbot
Com o workflow ativado no n8n e as credenciais configuradas, abra o seu bot no Telegram e faça os testes abaixo:

### ✅ Teste de Sucesso
Envie o nome de uma cidade seguindo o padrão **Cidade,UF,BR**.
* **Entrada:** `Itajaí, SC, BR` ou `São Paulo, SP, BR`
* **Retorno Esperado:** `Olá! Tudo pronto para o seu dia? Em Itajaí, a temperatura atual está em 18°C. Um friozinho agradável, ideal para um casaquinho!` *(Resposta gerada via Gemini)*.

### ❌ Teste de Tratamento de Erro
Envie um formato inválido, uma cidade que não existe, ou omita a sigla do país.
* **Entrada:** `CidadeFalsa, XYZ` ou apenas `Blumenau, SC`
* **Retorno Esperado:** `❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).`