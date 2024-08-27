# Documentação de Uso de Cookies no Projeto
Esta documentação explica como implementar e gerenciar o consentimento de cookies em seu projeto Next.js, utilizando um banner de consentimento e integrando o Google Analytics (GA4).
## Componentes Utilizados
1. **CookieConsentBanner.tsx:**
- Componente para exibir um banner solicitando o consentimento do usuário para o uso de cookies.
- Inclui um link para a Política de Privacidade.
2. **CookieConsentWrapper.tsx:**
- Componente para gerenciar o consentimento de cookies e condicionalmente renderizar o Google Analytics com base no consentimento.
3. **ClientAnalytics.tsx:**
- Componente para integrar o Google Analytics no cliente após o consentimento.
## Passos de Instalação
1. **Instalação de Dependências**
Para o gerenciamento de consentimento de cookies, é necessário instalar a biblioteca ``react-cookie-consent``:

```
yarn add react-cookie-consent
```
Para a integração do Google Analytics, utilize o pacote ``@next/third-parties``:
```
yarn add @next/third-parties
```
## Descrição dos Componentes
1. **Componente ``CookieConsentBanner.tsx``**
Este componente é responsável por exibir o banner de consentimento para o usuário. O banner só é exibido se o usuário ainda não tiver dado seu consentimento.
- **Props:**
  - `onConsent`: Função chamada quando o usuário aceita o uso de cookies.
- **Hooks:**
  - `useState`: Para controlar se o consentimento foi dado.
  - `useEffect`: Para verificar o status do consentimento no localStorage.
```
import React, { useState, useEffect } from 'react';
import CookieConsent from 'react-cookie-consent';
import Link from 'next/link';

const CookieConsentBanner: React.FC<{ onConsent: () => void }> = ({ onConsent }) => {
  const [consentGiven, setConsentGiven] = useState<boolean>(false);

  useEffect(() => {
    const userConsent = localStorage.getItem('user_consent');
    if (userConsent === 'true') {
      setConsentGiven(true);
    }
  }, []);

  const handleConsent = () => {
    localStorage.setItem('user_consent', 'true');
    setConsentGiven(true);
    onConsent();
  };

  if (consentGiven) return null;

  return (
    <CookieConsent
      location="bottom"
      buttonText="Aceitar todos os cookies"
      cookieName="user_consent"
      style={{ background: "#f4f4f4", color: "#000", fontSize: "13px" }}
      buttonStyle={{
          background: "#171717",
          color: "#fff",
          fontSize: "13px",
          display: "flex",
          justifyContent: "center",
          alignItems: "center",
          borderRadius: "5px" }}
      expires={150}
      onAccept={handleConsent}
    >
      Este site usa cookies para melhorar sua experiência,
      lembrando suas preferências e visitas.
      Ao continuar, você concorda com o uso de cookies.
      <Link
         href="/politica"
         target='_blank'
         rel="noopener noreferrer"
         className='font-semibold underline'>
         Politicas de Privacidade
      </Link>
    </CookieConsent>
  );
};

export default CookieConsentBanner;
```
2. **Componente ``CookieConsentWrapper.tsx``**
Este componente gerencia o estado do consentimento e exibe o componente de Analytics condicionalmente. Ele utiliza o useState para monitorar se o consentimento foi dado.

```
import React, { useState } from 'react';
import CookieConsentBanner from '@/components/CookieConsentBanner';
import dynamic from 'next/dynamic';

const DynamicClientAnalytics = dynamic(() => import('@/components/ClientAnalytics'), { ssr: false });

const CookieConsentWrapper: React.FC = () => {
  const [consentGiven, setConsentGiven] = useState<boolean>(false);

  const handleConsent = () => {
    setConsentGiven(true);
  };

  return (
    <>
      <CookieConsentBanner onConsent={handleConsent} />
      {consentGiven && <DynamicClientAnalytics />}
    </>
  );
};

export default CookieConsentWrapper;
```
3. **Componente ``ClientAnalytics.tsx``**
Este componente é responsável por inicializar o Google Analytics somente no cliente e após o consentimento.

```
import React from 'react';
import { GoogleAnalytics } from '@next/third-parties/google';

const ClientAnalytics: React.FC = () => {
  return <GoogleAnalytics gaId="Your_Measurement_ID" />;
};

export default ClientAnalytics;
```
## Integração no Layout Principal
Para integrar o sistema de consentimento ao layout principal do aplicativo, o componente `CookieConsentWrapper` é importado e utilizado no arquivo `app/layout.tsx`.

```
import type { Metadata } from "next";
import { Poppins } from "next/font/google";
import CookieConsentWrapper from '@/components/CookieConsentWrapper';


export const metadata: Metadata = {
  title: "Website",
  description: "Website Description",
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="pt-BR">
      <body>
        {children}
        <CookieConsentWrapper />
      </body>
    </html>
  );
}
```
## Como Funciona
1. **Banner de Consentimento:**
- Exibe um banner ao usuário solicitando consentimento para o uso de cookies.
- Armazena a escolha do usuário no ``localStorage``.
2. **Armazenamento do Consentimento:**
- O consentimento é salvo no ``localStorage`` com a chave ``user_consent``.
3. **Google Analytics:**
- Só é ativado se o usuário der consentimento.
- O componente `ClientAnalytics` é renderizado condicionalmente.
4. **Tailwind CSS:**
- Para utilizar Tailwind CSS em vez de estilos inline, você pode substituir as classes diretamente nas propriedades className do componente `CookieConsent`.
