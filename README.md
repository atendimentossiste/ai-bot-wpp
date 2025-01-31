# **SERVIDOR DE INTEGRAÇÃO WHATSAPP COM IA**
Este servidor Node.js permite a integração do WhatsApp Web com IA generativa da Google e armazenamento no MongoDB. Ele gerencia sessões de WhatsApp, captura QR Codes para autenticação e responde automaticamente a mensagens com base na IA.

---
### **CONFIGURAÇÃO INICIAL**
**INSTALANDO AS DEPENDÊNCIAS** <br>
<br>
Use o comando a seguir para instalar as dependências do projeto:
```bash 
npm install
```
**CONFIGURAÇÃO DO ARQUIVO ```.env```** <br>
<br>
Crie um arquivo ```.env``` na pasta /API e adicione as seguintes variáveis:
```bash 
API_KEY=AIzaSyDGvBXD3_YigImePumYQO1lYR16sCdT4rs
MONGO_URI=mongodb+srv://feliperibeirots:Lalelilolu07@cluster0.jdgyl.mongodb.net/atendimentos?retryWrites=true&w=majority&appName=Cluster0
```
---

### **FUNCIONALIDADES**
**1. Conexão com MongoDB**<br>
A função ```connectToMongoDB()``` estabelece a conexão com o banco de dados.
```bash 
async function connectToMongoDB() {
    try {
        await mongoose.connect(mongoURI);
        console.log('Conectado ao MongoDB');
    } catch (err) {
        console.error('Erro ao conectar ao MongoDB', err);
        process.exit(1); 
    }
}
```
**2. Gerenciamento de Clientes WhatsApp** <br>
Os clientes são armazenados na variável ```clients``` e podem ser gerenciados através das requisições HTTP.<br>
<br>
**3. QR Code para Autenticação**<br>
Quando um usuário se registra, um QR Code é gerado para autenticação no WhatsApp Web. <br>
**Rota:**
```bash 
GET /qr/:clientId
```
**Resposta esperada:**
```bash 
{
  "qrCode": "data:image/png;base64,..."
}
```

**4. Envio de Mensagens e Respostas com IA**<br>
Os usuários podem se registrar via ```API REST```, e sessões do WhatsApp Web são criadas automaticamente.<br>
**Rota:**
```bash
POST /user
```
**Requisição:**
```bash
{
  "nome": "João",
  "email": "joao@email.com",
  "senha": "123456",
  "numero": "+5511999999999",
  "descricao": "Atendimento ao cliente",
  "tipo_de_envio": "manual"
}
```
**Resposta esperada:**
```bash
{
  "id": "+5511999999999",
  "nome": "João",
  "email": "joao@email.com",
  "descricao": "Atendimento ao cliente",
  "tipo_de_envio": "manual",
  "qrCode": "data:image/png;base64,..."
}
```
**6. Publicação de Mensagens no Servidor de Atendimento**<br>
As mensagens recebidas são enviadas para um servidor externo para registro e análise.
```bash
const postMessage = async (clientId, message, numero, respostaIa) => {
    try {
        const user = clients[clientId];
        if (!user) {
            throw new Error('User not found');
        }
        const data = {
            mensagem: message,
            usuario_id: user.id,
            status_chat: true,
            nm_cliente: user.nome,
            nr_cliente: numero,
            reposta_ia: respostaIa,
        };
        const response = await axios.post('https://api-atendimentos.onrender.com/mensagens', data);
        return response;
    } catch (error) {
        console.error('Error posting message:', error);
        return null;
    }
}
```

---

### **SERVIDOR EXPRESS**<br>
<br>

O servidor Express gerencia requisições ```HTTP``` para interações com WhatsApp e IA.

```bash
const app = express();
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));
```

**Iniciar o Servidor:**
```bash
node server.js
```
