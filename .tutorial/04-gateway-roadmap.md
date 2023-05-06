# Gateway Roadmap

> Future developments in Fedimint Lightning Gateways.

- Dynamic and configurable fees for Lightning Gateways ([WIP](https://github.com/fedimint/fedimint/issues/524))

> Gateways are fair market competitors in providing services to Fedimint communities. Fees for each swap incentivize different gateway providers for their service. As such, we are working on dynamic fee models for gateway providers to build configure their services upon.

- Client upgrade for gateways ([WIP](https://github.com/fedimint/fedimint/pulls?q=is%3Apr+client-ng))

> The initial fedimint client falls just short of the requirements of a highly efficient gateway. It was built for serial processing of transactions and doesn't fully model atomicity in transaction states, including resume and revert of failed transactions.
> Gateways are getting an upgrade with a brand new fedimint client based on state machines. These will allow parallelization of transactions, useful for a single gateway that scales to serve large and multiple federations.

- Bolt 12 offers for Lightning Gateways (TBD)

> Going beyond Bolt 11 invoices for fedimint communities. We are [discussing future support](https://github.com/fedimint/fedimint/discussions/1507) for this new lightning spec for the benefit of federations.
>
> Additionally, there's room to [explore integrating LNURL systems to fedimint](https://github.com/fedimint/fedimint/discussions/1516) federations. Definitely related to swap functionality of a gateway

- Other gateway variants (TBD)

> Fedimint federation system can hold any number of assets, not just Bitcoin. If you built a fedimint module for such assets, youd need to specialized gateway as swap service providers. What would you like to see?
> 
>  - Taro asset gateways?
>  - Fiat gateways for bitcoin stability modules?
