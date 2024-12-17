# Cadenapp - Pivoting to crypto

## Summary

- [Context](#context)
- [The problem we're trying to solve](#the-problem-were-trying-to-solve)
- [Starting principles](#starting-principles)
- [Workflow](#workflow-for-abstracting-crypto-in-a-web-based-p2p-savings-platform)
- [Implementation Steps](#implementation-steps)
- [Benefits](#benefits-of-this-approach)
- [Challenges](#challenges-to-consider)
- [Solving the KYC challenge](#solving-the-kyc-challenge)


## Context
[Cadenapp](https://cadenapp.com/) is a web-based platform that enables users to securely form saving groups among peers, organize their savings collectively, and quickly distribute funds between each other. We built and deployed a web application ([Rails app](https://cadenapp.com/)) that allows members to join a saving group, organize a deposit calendar, define a withdrawal schedule, calculate saving goals, receive notifications, and more. We're tackling our final challenge before launch: handling transactions.

## The problem we're trying to solve
- We initially explored using a Banking-as-a-Service (BaaS) provider but found the costs prohibitive. The combination of very high service fees, the burden of their commercial terms, and the resources required to connect their services were too much for our available budget and operational needs.
- We're investigating achieving the same functionality at a much lower cost using a combination of crypto-to-fiat gateways and a custodial wallet. In a nutshell, user deposits fiat that is converted to a stablecoin, stablecoin deposits are agregated and then sold for fiat, fiat conversions are then transferred to bank accounts.

## Starting principles
- Matching the input and output methods (fiat-in/fiat-out OR crypto-in/crypto-out) is the best approach for user experience and simplicity.
- Keeping the end user experience fiat-focused avoids the need to explain blockchain concepts, wallets, or cryptocurrencies, and broadens the potential user base.
- Using crypto stablecoins to distribute funds internally avoids traditional banking fees and regulatory issues that are suboptimal for frequent micro-transactions in the context of a P2P saving platform.
- Using crypto in the backend outmatches the cost of traditional transactions, but converting to fiat has a cost (both on deposit and withdrawal). We need to find a balance between the two.
- One of the deciding factors for the success this pivot to crypto in the backend will be finding a player that can wire our crypto sales to Colombian bank accounts. Subsequentially chosing between [COPM](https://minteo.com/) and USDC is also key.

## Workflow for abstracting crypto in a web-based P2P savings platform

### 1.	User deposits fiat (using Fiat-to-Crypto gateway):
- Users make a fiat payment through a gateway
- The gateway converts the fiat payment to a stablecoin (USDC or COPM) and transfers it to a custodial wallet (explained below) owned by our platform.

### 2.	Hold funds in crypto:
- All users’ stablecoin deposits are collected in a platform-controlled wallet on the [Polygon](https://polygonscan.com/) network.
- These funds remain as stablecoin until it’s time for withdrawal.

### 3.	Aggregate and sell crypto for fiat:
- When it’s time to pay the designated member, the Rails app triggers a process to:
- Aggregate the stablecoin deposits from the platform wallet.
- Use a crypto-to-fiat service like [Circle](https://www.circle.com/) or a decentralized exchange ([Binance](https://binance.com/) or [Kraken](https://kraken.com/) with fiat off-ramps to convert the stablecoin back to fiat.

### 4.	Send Fiat to the designated member:
- The converted fiat funds are transferred to the designated member via a fiat payment gateway (most exchanges use PayPal, Stripe, or a direct bank transfer API like [Plaid](https://plaid.com/) or Wise).
- **It is of vital importance to find a platform that allows us to transfer to our users colombianbank accounts** 

## Detailed mplementation Steps
### 1. Fiat-to-Crypto gateway integration
- Use APIs from a fiat-to-crypto provider ([MoonPay](https://moonpay.com/), [Ramp](https://ramp.network/), [Transak](https://transak.com/)) to:
    - Accept fiat payments from users within the Rails app.
    - Immediately convert these to stablecoins like USDC/COPM on the Polygon network.
    - Deposit the stablecoins into a custodial platform-controlled wallet.
- Providers like [Onramper](https://onramper.com/) offer white-label solutions where users don’t see any branding from the fiat-to-crypto provider.
- Advanced APIs from providers like [Circle](https://www.circle.com/) allow to programmatically initiate fiat-to-crypto conversions without exposing users to third-party interfaces.
- Example flow:
  1. User enters payment amount (e.g., $50).
  2. Fiat-to-crypto gateway handles fiat processing and sends $50 worth of USDC or COPM to the platform wallet.

### 2. Platform wallet management
- We must maintain a centralized custodial wallet for each group or for the entire app. A custodial wallet is technically a normal crypto wallet but differs in terms of ownership and management : the platform holds private keys and manages funds on behalf of its users. This simplifies the user experience since they don’t need to manage crypto wallets themselves.
- Third-party services like [Fireblocks](https://www.fireblocks.com/), [BitGo](https://www.bitgo.com/), or [Venly](https://www.venly.io/) can be used for custodial wallet management. A custodial wallet can also be built using libraries like web3.js or ethers.js.
- The platform is responsible to track deposits and ensure the balance aligns with user payments. It should potentially receive withdrawal orders from the Rails app. Blockchain APIs like [Alchemy](https://www.alchemy.com/), [Infura](https://www.infura.io/), or [Moralis](https://developers.moralis.com/) can be used to track and manage user wallet balances on the Polygon network.

### 3. Stablecoin aggregation and fiat conversion
- At the end of a savings cycle, when a group deposits but one user withdraws, our app determines the total stablecoin balance owed to the designated member.
- We use an exchange or fiat off-ramp to convert that amount of stablecoins to fiat:
    - [Circle](https://www.circle.com/): Allows conversion of USDC to fiat and direct deposits into bank accounts.
    - Centralized Exchanges: Transfer USDC to an exchange (e.g., Binance, Kraken), sell it for fiat, and withdraw fiat in the step below.

### 4. Fiat payout to designated member (if needed)
- After the conversion, use a fiat payment gateway to pay the designated member. This phase may be done in step 3 depending on the provider. in the U.S. it would typically be:
    - Wise: Global bank transfers with low fees.
    - Stripe/PayPal: Instant payouts for users with accounts.
    - Plaid: ACH transfers.

## Suggested stack recap
- Fiat-to-Crypto gateway: ([MoonPay](https://moonpay.com/), [Ramp](https://ramp.network/), [Transak](https://transak.com/) or others.
- Custodial wallet: [Fireblocks](https://www.fireblocks.com/), [BitGo](https://www.bitgo.com/), or [Venly](https://www.venly.io/)
- Deposits tracking API: [Alchemy](https://www.alchemy.com/), [Infura](https://www.infura.io/), or [Moralis](https://developers.moralis.com/)
- Stablecoin management: [Polygon](https://polygonscan.com/)
- Crypto-to-Fiat conversion: [Circle](https://www.circle.com/), [Binance](https://binance.com/) or [Kraken](https://kraken.com/)
- Fiat Payouts: _to be investigated / defined_ (in the U.S. it could be Stripe, PayPal, or Wise)

## Benefits of this approach
1. Users see only fiat transactions: users deposit fiat and receive fiat, without ever interacting with or seeing crypto.
2. Simplifies UX: no need for users to understand crypto wallets, stablecoins, or blockchain transactions.
3. Reduces volatility/devaluation risks: using stablecoins like USDC avoids crypto price volatility / COP devaluation.
4. Minimizes fees: by pooling transactions, we reduce the number of on-chain operations, lowering fees.

## Challenges to consider
1. Compliance & regulation: fiat-to-crypto gateways and crypto-to-fiat services often require KYC/AML compliance. See more below.
2. Liquidity timing: the fiat conversion process may introduce delays depending on the gateway used. Ensure payouts align with user expectations.
3. Custodial Risks: holding funds in a centralized platform wallet introduces potential security risks. Consider integrating additional security measures like multi-signature wallets.
4. Transaction costs: while crypto reduces some fees, fiat off-ramping and bank payouts may still have associated costs.
5. Reliance on fiat-to-crypto gateways means our system depends on their uptime and fees.
6. During the sell phase (crypto-to-fiat), while the risk of conversion issues with small amounts is minimal, exchanges may request documentation about the origin and purpose of funds and temporarily block transfers to comply with regulations.

## Solving the KYC challenge
If we implement a custodial wallet and handle fiat-to-crypto conversions, we will likely need to comply with KYC (Know Your Customer) and AML (Anti-Money Laundering) regulations. Fiat-to-Crypto gateways are legally required to ensure that users are legitimate and not engaging in fraud, money laundering, or other illegal activities. To perform this check, gateways typically require user data like: full name, address, date of birth, government-issued ID (passport, driver’s license, etc.). If our app acts as the intermediary, we must collect and securely transmit this information to the gateway to facilitate transactions.

### How this works in practice
1. User provides data during registration or payment: KYC verification is integrated as part of the onboarding process or at the time of the first fiat-to-crypto transaction.
2. Data sent to fiat-to-crypto gateway: the API or SDK of the gateway is used to send the user’s data securely for verification.
3. Gateway handles compliance: the gateway will validate the user’s identity and flag potential risks.
4. Result: once the user’s identity is verified, the gateway allows transactions to proceed.

### Options for Managing KYC/AML
##### *Option 1:* Delegate to fiat-to-crypto gateway
- Use a third-party gateway that includes KYC/AML checks.
- The gateway performs verification directly with users and manages compliance.
- Advantages: reduces liability and operational burden, minimal implementation effort on our end (just an API or widget integration).
- Disadvantages: we must share user data with the gateway, users might experience delays due to verification checks.

##### *Option 2*: Build a proprietary KYC process
- Build and manage a KYC/AML verification system in our app.
- Use third-party tools like [SumSub](https://sumsub.com/), Jumio, or Onfido to verify user identities.
- Advantages: complete control over user data and branding, can integrate the KYC process directly into app for a seamless experience.
- Disadvantages: Expensive to implement and maintain, directly liable for compliance with local laws.
