# Integração com o WhatsApp

Este documento descreve como a integração com o WhatsApp funciona no aplicativo BuscaSim.

## Configuração

A integração com o WhatsApp utiliza a API do WhatsApp Business via Facebook Graph API. Para configurá-la, você precisa definir as seguintes variáveis de ambiente na página de configurações:

```
WHATSAPP_PHONE_NUMBER_ID=seu_phone_number_id
WHATSAPP_API_TOKEN=seu_api_token
WHATSAPP_BUSINESS_ACCOUNT_ID=seu_id_conta_business
WHATSAPP_WEBHOOK_VERIFY_TOKEN=token_que_sera_adicionado_no_painel_desenvolvedor_do_facebook
```

## Como Funciona

1. O aplicativo permite o envio de mensagens do WhatsApp para clientes em massa
2. As mensagens são enviadas através da API do WhatsApp Business
3. A implementação usa uma fila para gerenciar o envio de mensagens de forma assíncrona
4. Dois tipos de mensagens são suportados:
   - Mensagens personalizadas (mais flexíveis, mas limitadas a usuários que deram consentimento)
   - Mensagens modelo (modelos pré-aprovados que podem ser enviados para qualquer usuário)

## Detalhes da API

A integração utiliza o endpoint da Facebook Graph API:
```
https://graph.facebook.com/v22.0/{phone-number-id}/messages
```

## Implementação

- Em `Pedidos->(Selecionar pedidos via checkboxes)->Ações em massa (n)->Enviar Whatsapp`: Permite enviar mensagens do WhatsApp (personalizadas e modelo) para vários clientes

## Mensagens Modelo

As mensagens modelo são necessárias quando:
1. Você deseja iniciar conversas com usuários que não enviaram mensagens nas últimas 24 horas
2. Você deseja enviar conteúdo promocional ou de marketing

Os modelos devem ser:
1. Criados e aprovados no WhatsApp Business Manager

## Como Criar e Usar Modelos

1. Em `Templates Whatsapp` Clique em `Criar Novo Template` (Nome do Template e Conteúdo do Corpo na aba Corpo são obrigatórios)
2. Aguarde a aprovação (normalmente entre 24-48 horas)
3. Use o modelo aprovado no aplicativo, selecionando-o no a aba Modelo Pré aprovado ao enviar mensagens em massa

## Observações

- Se token da API não for do tipo System User Access (gerado diretamente em `API Setup` em https://developers.facebook.com/) ele possuirá uma data de expiração e precisará ser atualizado periodicamente
- Para criar um token de API do tipo System User Access acesse https://business.facebook.com/ e em `configurações->Users->System users` crie um usuário de sistema se ainda não existir
- Ao lado de Revoke tokens clique no icone de menu e em Assign assets na seção de apps verifique se o app responsável pela integração com o whatsapp business está selecionado e as permissões habilitadas
- Falhas no envio das mensagens são registradas para fins de depuração
- Mensagens modelo podem ser enviadas para qualquer usuário, mas mensagens personalizadas só podem ser enviadas para usuários que tenham enviado mensagens nas últimas 24 horas

## Configuração do webhook

Para que os modelos de mensagem sejam liberados após serem aprovados pela Meta é necessário fazer os seguintes passos no painel de desenvolvedor

1. Acesse o painel de desenvolvedor do Facebook em https://developers.facebook.com/apps/
2. Selecione o app responsável pela integração com o WhatsApp Business
  ![Passo 1](https://i.postimg.cc/NGNkPJdK/webhook-1.png)

3. No menu lateral esquerdo clique em WhatsApp -> Configuração
  ![Passo 1](https://i.postimg.cc/fTJf4cPB/webhook-2.png)

4. Na seção Webhooks preencha os campos:
   - Callback URL: URL pública do seu servidor onde o webhook será processado (https://buscasim.com.br/api/whatsapp-webhook)
   - Token de verificação: O mesmo valor definido em WHATSAPP_WEBHOOK_VERIFY_TOKEN na página de configurações https://buscasim.com.br/app/configuracoes

5. Após preencher, clique em Verify and save para que a Meta verifique se está tudo ok e salve os dados

5. Encontre o item "message_template_status_update" clique em "Teste" para verificar se está tudo ok e ative esta opção
  ![Passo 1](https://i.postimg.cc/QCng9CFQ/webhook-3.png)

## Solução de Problemas

Se as mensagens não estiverem sendo entregues:

1. Verifique os logs do Laravel para erros na API
2. Confirme se o token da API do WhatsApp Business é válido e não está expirado
3. Garanta que os números de telefone estejam formatados corretamente
4. Verifique se a conta do WhatsApp Business está configurada corretamente e ativa
5. Para mensagens personalizadas, certifique-se de que o usuário tenha enviado uma mensagem nas últimas 24 horas
