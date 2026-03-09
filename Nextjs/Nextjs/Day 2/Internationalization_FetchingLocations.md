# **Internationalization (i18n) in Next.js**

## **Internationalization Concepts**
Internationalization (i18n) is the process of designing and preparing your application to support multiple languages and regions. Localization (l10n) is the actual adaptation of content for specific locales. Next.js provides built-in support for i18n through routing strategies, either using sub-path routing (`/en/about`) or domain routing (`en.example.com`).

**Key Components:**
- **Locale**: Language and region code (en-US, fr-FR, ja-JP)
- **Translations**: Mapping of keys to locale-specific strings
- **Routing Strategy**: How locales are represented in URLs
- **Locale Detection**: Automatic or manual language selection

```javascript
// next.config.js - Basic i18n setup
module.exports = {
  i18n: {
    locales: ['en', 'fr', 'de', 'ja'], // Supported languages
    defaultLocale: 'en', // Fallback language
    localeDetection: true, // Auto-detect user's language
  },
};
```

## **Client-Side Internationalization**
Client-side i18n handles translations in the browser, ideal for dynamic content and interactive components. It requires sending translation files to the client and managing locale state. Libraries like `next-i18next` or `react-i18next` are commonly used.

**Pros:** Dynamic updates, no server dependency
**Cons:** Larger initial bundle, SEO challenges

```jsx
// Client component with i18n
'use client';
import { useTranslation } from 'next-i18next';
import { useRouter } from 'next/router';

export default function ClientComponent() {
  const { t, i18n } = useTranslation('common');
  const router = useRouter();
  
  const changeLanguage = (locale) => {
    i18n.changeLanguage(locale);
    // Update URL without full page reload
    router.push(router.pathname, router.asPath, { locale });
  };
  
  return (
    <div>
      <h1>{t('welcome')}</h1>
      <p>{t('description')}</p>
      
      <select 
        value={i18n.language} 
        onChange={(e) => changeLanguage(e.target.value)}
      >
        <option value="en">English</option>
        <option value="fr">Français</option>
        <option value="de">Deutsch</option>
      </select>
    </div>
  );
}

// Example translation file (public/locales/en/common.json)
{
  "welcome": "Welcome to our site!",
  "description": "This is a client-side translated component."
}
```

## **Server-Side Internationalization**
Server-side i18n happens during server rendering, providing better SEO and performance. Next.js's built-in i18n routing automatically handles locale prefixes and provides locale information to both pages and API routes.

**Pros:** Better SEO, smaller client bundle, faster initial load
**Cons:** Requires server-side rendering, more complex setup

```jsx
// Server Component with i18n (App Router)
import { getTranslations } from 'next-intl/server';
import { setRequestLocale } from 'next-intl/server';

// This runs on the server
export default async function ServerPage({ params: { locale } }) {
  // Set locale for this request
  setRequestLocale(locale);
  
  // Get translations on server
  const t = await getTranslations('common');
  
  // Fetch locale-specific data
  const localeData = await fetchData(locale);
  
  return (
    <div>
      <h1>{t('welcome')}</h1>
      <p>{localeData.description}</p>
      
      {/* Language switcher with server-side navigation */}
      <div className="language-switcher">
        <a href="/en/about">English</a>
        <a href="/fr/about">Français</a>
        <a href="/de/about">Deutsch</a>
      </div>
    </div>
  );
}

// next-intl configuration (app/[locale]/layout.tsx)
import { NextIntlClientProvider } from 'next-intl';
import { getMessages } from 'next-intl/server';

export default async function LocaleLayout({
  children,
  params: { locale }
}: {
  children: React.ReactNode;
  params: { locale: string };
}) {
  const messages = await getMessages();
  
  return (
    <NextIntlClientProvider messages={messages}>
      {children}
    </NextIntlClientProvider>
  );
}
```

## **Working with Data in i18n Applications**
Localized applications need to handle locale-specific data fetching, formatting, and storage.

### **1. Database Design for Multi-language Content**
```sql
-- Example schema for localized content
CREATE TABLE products (
  id UUID PRIMARY KEY,
  base_price DECIMAL
);

CREATE TABLE product_translations (
  product_id UUID REFERENCES products(id),
  locale VARCHAR(5),
  name VARCHAR(255),
  description TEXT,
  PRIMARY KEY (product_id, locale)
);
```

### **2. Fetching Locale-Specific Data**
```typescript
// Data fetching utility
interface LocalizedData<T> {
  [locale: string]: T;
}

async function fetchLocalizedData<T>(
  endpoint: string,
  locale: string
): Promise<T> {
  // Strategy 1: Separate endpoints
  const response = await fetch(`${endpoint}?locale=${locale}`);
  
  // Strategy 2: Single endpoint with Accept-Language header
  const response2 = await fetch(endpoint, {
    headers: {
      'Accept-Language': locale,
    },
  });
  
  // Strategy 3: GraphQL with locale argument
  const graphqlResponse = await fetch('/api/graphql', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      query: `
        query GetProduct($id: ID!, $locale: String!) {
          product(id: $id) {
            id
            name(locale: $locale)
            description(locale: $locale)
          }
        }
      `,
      variables: { id: '123', locale },
    }),
  });
  
  return response.json();
}

// Server Component fetching
export default async function ProductPage({ params: { locale, id } }) {
  // Fetch both product data and translations
  const [product, translations] = await Promise.all([
    fetchProduct(id),
    fetchProductTranslations(id, locale),
  ]);
  
  return (
    <div>
      <h1>{translations.name || product.defaultName}</h1>
      <p>{translations.description || product.defaultDescription}</p>
      <p>{formatCurrency(product.price, locale)}</p>
    </div>
  );
}
```

## **Fetching Locations (Geolocation with i18n)**
Combining location data with internationalization for personalized experiences.

### **1. Server-Side Location Detection**
```javascript
// middleware.js - Detect location and set locale
import { NextResponse } from 'next/server';
import { match } from '@formatjs/intl-localematcher';
import Negotiator from 'negotiator';

const locales = ['en', 'fr', 'de', 'ja'];
const defaultLocale = 'en';

// Get locale from headers
function getLocale(request) {
  const headers = { 'accept-language': request.headers.get('accept-language') || '' };
  const languages = new Negotiator({ headers }).languages();
  
  return match(languages, locales, defaultLocale);
}

export function middleware(request) {
  // Check if locale is already in path
  const { pathname } = request.nextUrl;
  const pathnameHasLocale = locales.some(
    locale => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  );
  
  if (pathnameHasLocale) return NextResponse.next();
  
  // Get user's locale from browser or geolocation
  const locale = getLocale(request);
  
  // Redirect to locale-prefixed path
  request.nextUrl.pathname = `/${locale}${pathname}`;
  return NextResponse.redirect(request.nextUrl);
}

// API route with location-based content
export async function GET(request) {
  const { geo } = request;
  const country = geo?.country || 'US';
  const city = geo?.city || 'Unknown';
  
  // Get locale based on country
  const locale = getLocaleFromCountry(country);
  
  // Fetch location-specific data
  const weather = await fetchWeather(city, locale);
  const localNews = await fetchLocalNews(country, locale);
  
  return Response.json({
    location: { country, city },
    locale,
    weather,
    news: localNews,
  });
}
```

### **2. Client-Side Geolocation with i18n**
```jsx
'use client';
import { useEffect, useState } from 'react';
import { useRouter } from 'next/navigation';

export default function LocationAwareComponent() {
  const [location, setLocation] = useState(null);
  const [locale, setLocale] = useState('en');
  const router = useRouter();
  
  useEffect(() => {
    // Get browser location (requires user permission)
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        async (position) => {
          const { latitude, longitude } = position.coords;
          
          // Reverse geocode to get country/city
          const locationData = await fetch(
            `https://api.bigdatacloud.net/data/reverse-geocode-client?latitude=${latitude}&longitude=${longitude}&localityLanguage=${locale}`
          ).then(res => res.json());
          
          setLocation(locationData);
          
          // Update locale based on country
          const detectedLocale = getLocaleFromCountry(locationData.countryCode);
          if (detectedLocale !== locale) {
            setLocale(detectedLocale);
            router.push(`/${detectedLocale}/local-content`);
          }
        },
        (error) => {
          console.error('Geolocation error:', error);
        }
      );
    }
  }, [locale, router]);
  
  return (
    <div>
      {location && (
        <div>
          <h2>{locale === 'en' ? 'Your Location' : 
                locale === 'fr' ? 'Votre Emplacement' : 
                locale === 'de' ? 'Ihr Standort' : '位置情報'}</h2>
          <p>{location.city}, {location.countryName}</p>
          {/* Show location-specific content */}
        </div>
      )}
    </div>
  );
}
```

## **Advanced i18n Patterns**

### **1. RTL (Right-to-Left) Language Support**
```jsx
// Layout component that detects RTL languages
export default function InternationalLayout({ children, locale }) {
  const isRTL = ['ar', 'he', 'fa', 'ur'].includes(locale);
  
  return (
    <html dir={isRTL ? 'rtl' : 'ltr'} lang={locale}>
      <body className={isRTL ? 'rtl-styles' : 'ltr-styles'}>
        {children}
      </body>
    </html>
  );
}

// CSS with RTL support
.rtl-styles {
  text-align: right;
  margin-left: auto;
  margin-right: 0;
}

.ltr-styles {
  text-align: left;
  margin-left: 0;
  margin-right: auto;
}
```

### **2. Dynamic Import of Translation Files**
```javascript
// Dynamic import based on locale (reduces bundle size)
async function loadTranslations(locale) {
  switch (locale) {
    case 'fr':
      return await import('@/locales/fr.json');
    case 'de':
      return await import('@/locales/de.json');
    default:
      return await import('@/locales/en.json');
  }
}

// Usage in component
'use client';
import { useEffect, useState } from 'react';

export default function DynamicTranslations({ locale }) {
  const [translations, setTranslations] = useState({});
  
  useEffect(() => {
    loadTranslations(locale).then(setTranslations);
  }, [locale]);
  
  return <div>{translations.welcome}</div>;
}
```

### **3. SEO Optimization for i18n**
```jsx
// Add hreflang tags for SEO
export const metadata = {
  alternates: {
    canonical: 'https://example.com',
    languages: {
      'en-US': 'https://example.com/en',
      'fr-FR': 'https://example.com/fr',
      'de-DE': 'https://example.com/de',
      'ja-JP': 'https://example.com/ja',
    },
  },
  openGraph: {
    locale: 'en_US',
    // Generate locale-specific Open Graph data
  },
};

// Generate sitemap with locales
export async function generateSitemaps() {
  return [
    { id: 'en', url: '/sitemap/en.xml' },
    { id: 'fr', url: '/sitemap/fr.xml' },
    { id: 'de', url: '/sitemap/de.xml' },
  ];
}
```

## **Best Practices for i18n**

1. **Use a consistent key naming convention** across all translation files
2. **Separate translations by domain/feature** (common.json, auth.json, products.json)
3. **Handle pluralization and gender correctly** using ICU message format
4. **Store user locale preference** in cookies or database
5. **Test with pseudo-localization** to catch UI layout issues
6. **Consider date, time, and number formatting** differences
7. **Implement fallback strategies** for missing translations
8. **Monitor translation coverage** and missing keys

## **Popular i18n Libraries for Next.js**

1. **next-intl** - Recommended for App Router, React Server Components
2. **next-i18next** - Based on i18next, good for Pages Router
3. **react-intl** - Format.js implementation, enterprise-ready
4. **lingui** - Focus on developer experience with CLI tools

Choosing the right library depends on your rendering strategy (App Router vs Pages Router) and specific requirements like RTL support, pluralization, and ICU message format compatibility.