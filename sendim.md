# Presentation

Sending emails from a backend is a pretty common feature to have, so we, [at Flexper](https://flexper.hashnode.dev/), decided to make a library to simplify how to implement it inside our NodeJS applications.

In this article, we will present and use [sendim](https://github.com/flexper/sendim), our custom abstraction to wrap **any email providers** to send **raw and transactional emails** to your clients.

We have already made two adapters for the most common providers, [sendim-mandrill](https://github.com/flexper/sendim-mandrill) and [sendim-sendinblue](https://github.com/flexper/sendim-sendinblue)

Having the same interface for sending emails no matter the provider lets us easily swap between one another inside a project but also unifies how we are sending emails throughout every project.

# General use

As we said, the point of this library is to unify how to use email providers with a **general interface**.

From here, **no matter what adapter you will use, it will always looks the same**

```bash
npm i sendim
```

## Adapters

We have already made some adapters for the email providers we use.

We will see how to create your own adapters further down.

### Mandrill adapter

Here is how the configuration for using Mandrill as a provider looks like

```bash
npm i sendim-mandrill
```

```typescript
import { Sendim } from 'sendim';
import { SendimMandrillProviderConfig, SendimMandrillProvider } from 'sendim-mandrill';

const sendim = new Sendim();

await sendim.addTransport<SendimMandrillProviderConfig>(
  SendimMandrillProvider,
  { apiKey: process.env.MANDRILL_APIKEY! }, // You need to have this environment variable set in a .env file
);
```

### Sendinblue adapter

And now look how similar the Sendinblue adapter is

```bash
npm i sendim-sendinblue
```

```typescript
import { Sendim } from 'sendim';
import { SendimSendinblueProviderConfig, SendimSendinblueProvider } from 'sendim-sendinblue';

const sendim = new Sendim();

await sendim.addTransport<SendimSendinblueProviderConfig>(
  SendimSendinblueProvider,
  { apiKey: process.env.SENDINBLUE_APIKEY! }, // You need to have this environment variable set in a .env file
);
```

> You can add multiple adapters to the same 'Sendim' instance.

## Sending emails

### Strongly typed interface

Sendim is using very simple yet exhaustive interfaces in TypeScript in order to handle emailing.

Whether for sending transactional or raw emails, you'll be using the same general typing

```typescript
type MailOptions = {
  to: EmailInformation[];
  cc?: EmailInformation[];
  bcc?: EmailInformation[];

  sender: EmailInformation;
  reply?: EmailInformation;
  attachments?: EmailAttachment[];
};

interface EmailInformation {
  email: string;
  name?: string;
}

interface EmailAttachment {
  name: string;
  contentType: string;
  content: string;
}
```

Even tho you can add **multiple transports** *(adapters)* to the same *Sendim* instance, **by default, only the first added one will be used**. But you can specify a name as second argument of the commands below to **target another adapter** **('mandrill' or 'sendinblue')**.

### Raw emails

```typescript
await sendim.sendRawMail({
  // Common commands attributes
  to: [
    {
      email: 'test@test.fr',
    },
  ],
  sender: {
    email: 'test@test.fr',
  },

  // Raw mail attributes
  subject: 'How great is Sendim',
  htmlContent: '<b>Very much !</b>'
});
```

Instead of the *htmlContent* attribute you could use the *textContent* one

### Transactional emails

```typescript
await sendim.sendTransactionalMail({
  // Common commands attributes
  to: [
    {
      email: 'test@test.fr',
    },
  ],
  sender: {
    email: 'test@test.fr',
  },

  // Transactional mail attributes
  templateId: '6', // The template id/slug from your email provider
  params: { // An object to fill the variables in your template
    akey: 'My first value',
    anotherKey: 'My second value',
  },
});
```

# Create your own adapter

Here we will see the bases to create a **Sendim** adapter.

```typescript
import {
  RawMailOptions,
  SendimTransportInterface,
  TransactionalMailOptions,
} from 'sendim';

// That's the interface for how to initialize your adapter
export interface MyProviderConfig {
  apiKey: string;
}

// This is the adapter that you will add as a transport to yor future Sendim instance
export class MyProvider implements SendimTransportInterface {
  // You have to provide a name in order to be able to target it if your app is using multiple transports
  providerName = 'my-provider';

  constructor(public config: MyProviderConfig) {
    // Use your configuration interface to connect to your email provider: set class attributes, do requests ... 

    // ...
  }

  async isHealthy() {
    // You need to provide a way to check whether or not the connection to your provider is working
  }

  async sendRawMail({ attachments, ...options }: RawMailOptions) {
    // Implement the logic of your provider given the RawMailOptions interface common to every transports
  }

  async sendTransactionalMail({
    attachments,
    ...options
  }: TransactionalMailOptions) {
    // Implement the logic of your provider given the TransactionalMailOptions interface common to every transports
  }
}
```

# Conclusion

The Sendim library and its providers are a convenient way to deal with email without having to pay too much attention to the provider's implementation; Once abstracted in a Sendim adapter, you can have the same interface throughout your projects and easily switch them making your code more flexible

Feel free to [submit issues on the GitHub](https://github.com/flexper/sendim) if you're having any trouble and happy coding!
