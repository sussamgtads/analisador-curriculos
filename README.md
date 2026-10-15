# Analisador de Currículos

Site estático onde a pessoa sobe um currículo em PDF ou DOCX e recebe:

- nota geral (0–100)
- pontos positivos e pontos negativos
- chance de o texto ter sido gerado por IA, com os trechos suspeitos
- nome, e-mail e telefone do candidato
- veredito: vale a pena conversar ou não, com perguntas para a conversa
- **aderência à vaga** (opcional): o RH escreve os requisitos e a IA cruza com o currículo, listando o que atende, o que atende em parte e o que não atende. Requisitos discriminatórios (idade, gênero, etc.) são filtrados no workflow e ignorados.

Frontend em HTML/JS puro (GitHub Pages) + backend em **n8n** que extrai o texto do PDF e chama a **OpenAI**.

```
Navegador ──PDF/texto──▶ Webhook n8n ─▶ Extrai texto (ou OCR via OpenAI) ─▶ OpenAI (JSON) ─▶ Responde ─▶ Navegador
```

## Como ligar o backend

1. No n8n: **Workflows → Import from file** e escolha `n8n/workflow.json`.
2. No nó **OpenAI**, selecione (ou crie) a credencial *OpenAI API* com a sua chave.
3. **Ative** o workflow (Active) e copie a **Production URL** do nó Webhook
   (algo como `https://seu-n8n.com/webhook/analisar-curriculo`).
4. Cole essa URL em `config.js`:
   ```js
   window.APP_CONFIG = { WEBHOOK_URL: "https://seu-n8n.com/webhook/analisar-curriculo" };
   ```
5. Faça commit e push. O GitHub Pages atualiza sozinho.

Sem `WEBHOOK_URL` o site roda em **modo demo** com um resultado de exemplo.

## Rodar local

```bash
python3 -m http.server 8000   # abra http://localhost:8000
```

## Avisos

- A chave da OpenAI fica só no n8n, nunca no site.
- A URL do webhook é pública: quem a tiver pode gastar a sua cota. Considere limitar a taxa no n8n/proxy.
- Currículos têm dados pessoais (LGPD): informe os candidatos de que o arquivo é enviado à OpenAI e não guarde os dados sem necessidade.
- A detecção de IA é um **indício**, não prova. A nota é apoio à decisão; não use como único critério de seleção.
- **Formatos:** PDF com texto, PDF escaneado e DOCX. No DOCX o texto é extraído no navegador (mammoth.js). No PDF escaneado, o n8n detecta a falta de texto e envia o PDF direto ao modelo da OpenAI, que lê as imagens (OCR) — custa mais tokens. Arquivos `.doc` antigos não são suportados.
- Se a OpenAI recusar o PDF no nó HTTP, confirme que o modelo é de visão (`gpt-4o-mini` ou `gpt-4o`).
