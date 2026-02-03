## Create a fallback Retry-Provider for JsonRpcProvider

_Originally published on Aug 10, 2023 [here](https://blog.blockmagnates.com/create-a-fallback-retry-provider-for-jsonrpcprovider-ca0b35165788)._

---

The JsonRpcProvider class from Ethers.js provides a convenient interface for making JSON-RPC calls to the Ethereum network, regardless of whether we're connecting to our own node or using a third-party service like *Infura*.

However, sometimes the third-party services that provide URLs for connections are unreliable or we might hit their rate limits. In order to avoid that, we can implement our own custom class that extends JsonRpcProvider.

Let's call our custom class RetryProvider. We can make it retry capabilities as well as fallback URLs, thus increasing the likelihood that our transactions get carried through.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*a95Ftz4fS-bBJ3oHq_BGPA.png)

### Creating RetryProvider

Here is what it could look like, the explanation is below:

```react
import { Logger } from '@ethersproject/logger'
import { defineReadOnly } from '@ethersproject/properties'
import { JsonRpcProvider } from '@ethersproject/providers'

import { MAX_PROVIDER_RETRIES, RETRY_PROVIDER_INTERVAL } from './constants'

export class RetryProvider extends JsonRpcProvider {
  readonly provider: JsonRpcProvider
  readonly fallbackProvider: JsonRpcProvider
  readonly logger: Logger

  constructor(provider: JsonRpcProvider, fallbackProvider: JsonRpcProvider) {
    super()

    defineReadOnly(this, 'provider', provider)
    defineReadOnly(this, 'fallbackProvider', fallbackProvider)
    defineReadOnly(this, 'logger', new Logger('RetryProvider'))
  }

  async perform(method: string, params: any): Promise<any> {
    let useFallbackNode = false
    let retries = 0
    let response
    let errorCode

    while (retries <= MAX_PROVIDER_RETRIES) {
      try {
        response = useFallbackNode
          ? await this.provider.perform(method, params)
          : await this.fallbackProvider.perform(method, params)
        break
      } catch (error: any) {
        if (error?.body) {
          errorCode = JSON.parse(error.body).error?.code
        } else {
          errorCode = error.code
        }

        if ((errorCode === 429 || errorCode >= 500) && retries < MAX_PROVIDER_RETRIES) {
          this.logger.warn(`Retrying request with other provider.`)
          useFallbackNode = !useFallbackNode
          retries++
          await new Promise((resolve) => setTimeout(resolve, RETRY_PROVIDER_INTERVAL))
        } else {
          this.logger.throwError(`Provider error not recognized: ${JSON.stringify(error)}`)
          break
        }
      }
    }

    if (!response) {
      this.logger.throwError(`No response from provider.`)
    }

    return response
  }
}
```

The RetryProvider is designed to automatically retry failed requests to a JSON-RPC provider, such as Infura, using a fallback provider.

The RetryProvider constructor takes two JsonRpcProvider instances as arguments: the primary `provider` and the fallback `fallbackProvider`. The `perform` method of the RetryProvider class overrides the `perform` method of the JsonRpcProvider class to include the retry logic.

The retry logic involves trying to execute the method using the primary `provider`. If the primary `provider` fails with an error code of 429 (too many requests) or 5xx (server error), then it tries again with the `fallbackProvider`. The retrying process can occur up to a maximum number of times as specified by the `MAX_PROVIDER_RETRIES` constant. If all retries fail, the method throws an error.

### Using RetryProvider

Here is a simple test that demonstrates how the RetryProvider could be used:

```react
import { RetryProvider } from './RetryProvider';

describe('RetryProvider', () => {

  test('should successfully send a JSON-RPC request after retries', async () => {
    const brokenNodeUrl = 'https://endpoint-1.example.com'; // Make sure that this nodeURL doesn't work
    const workingNodeUrl = 'https://endpoint-2.example.com';
    const provider = new RetryProvider(brokenNodeUrl, workingNodeUrl);

    const blockNumber = await provider.getBlockNumber();
    expect(blockNumber).toBeGreaterThan(0);
  });

  test('should fail to send a JSON-RPC request after max retries', async () => {
    const brokenNodeUrl = 'https://endpoint-1.example.com'; // Make sure that this nodeURL doesn't work
    const alsoBrokenNodeUrl = 'https://endpoint-2.example.com'; // Also make sure that this nodeURL doesn't work
    const provider = new RetryProvider(brokenNodeUrl, alsoBrokenNodeUrl);

    await expect(provider.getBlockNumber()).toThrow();
  });
});
```

### Conclusion

The example of a RetryProvider in this article shows a convenient way to use a fallback provider automatically in case the primary provider fails, allowing for a more reliable and robust connection to the Ethereum network.

If it was helpful, please click the clap 👏 button.
And feel free to comment! I'd be happy to help :)
