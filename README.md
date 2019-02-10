# Azure VM pricing

Mass-pricing of `VMs` on `Azure` based on `CPU` cores count and memory. This is useful when costing a lift-and-shift migration dealing with many thousands `VMS` of varied sizes.

The pricing is retrieved from [Virtual Machines Pricing][virtual-machines-pricing].

:rotating_light: This tool will only provide you with an **estimation**. Depending on your `Azure` spends you might be able to get a better deal from `Microsoft`. You should use the output of this tool as a coarse-grain estimation. On top of the `VM` price you will also need to consider [storage][managed-disks-pricing] and [egress][bandwidth-pricing-details] costs.

This tool is composed of two components:

1. A [Parser](#parser) retrieving the pricing from [Virtual Machines Pricing][virtual-machines-pricing]
2. A [Coster](#coster) using the output from the `Parser` and a list of `VMs` specifications to establish their price

This approach allows to decouple pricing acquisition from its usage and open the door to automation. The `Parser` can be scheduled to retrieve the pricing at regular interval and the `Coster` can then use an always up-to-date pricing.

## Parser

Retrieve `VMs` **hourly** pricing for a specific **culture**, **currency**, **operating system** and **region**.

| Culture | Culture display name | Currency | Currency display name | Support |
| - | - | - | - | - |
| `en-us` | `English (US)` | `USD` | `US Dollar ($)` | :white_check_mark: |
| | | `SAR`[[8]](#closest-currency-8) | `Saudi Riyal (SR)` | :white_check_mark: |
| `cs-cz` | `Čeština` | `EUR`[[1]](#closest-currency-1) | `Euro (€)` | :white_check_mark: |
| `da-dk` | `Dansk` | `DKK` | `Danish Krone (kr)` | :white_check_mark: |
| `de-de` | `Deutsch` | `EUR` | `Euro (€)` | :white_check_mark: |
| | | `CHF`[[9]](#closest-culture-9)  | `Swiss Franc. (chf)` | :white_check_mark: |
| `en-au` | `English (Australia)` | `AUD` | `Australian Dollar ($)` | :white_check_mark: |
| `en-ca` | `English (Canada)` | `CAD` | `Canadian Dollar ($)` | :white_check_mark: |
| `en-in` | `English (India)` | `INR` | `Indian Rupee (₹)` | :white_check_mark: |
| `en-gb` | `English (UK)` | `GBP` | `British Pound (£)` | :white_check_mark: |
| | | `MYR`[[6]](#closest-culture-6) | `Malaysian Ringgit (RM$)` | :white_check_mark: |
| | | `ZAR`[[4]](#closest-culture-4) | `South African Rand (R)` | :white_check_mark: |
| | | `NZD`[[7]](#closest-culture-7) | `New Zealand Dollar ($)` | :white_check_mark: |
| | | `HKD`[[10]](#closest-culture-10) | `Hong Kong Dollar (HK$)` | :white_check_mark: |
| `es-es` | `Español` | `EUR` | `Euro (€)` | :white_check_mark: |
| | | `ARS`[[5]](#closest-culture-5) | `Argentine Peso ($)` | :white_check_mark: |
| `es-mx` | `Español (MX)` | `MXN` | `Mexican Peso (MXN$)` | :white_check_mark: |
| `fr-fr` | `Français` | `EUR` | `Euro (€)` | :white_check_mark: |
| | | `CHF`[[9]](#closest-culture-9)  | `Swiss Franc. (chf)` | :white_check_mark: |
| `fr-ca` | `Français (Canada)` | `CAD` | `Canadian Dollar ($)` | :white_check_mark: |
| `is-is` | `Íslensku` | `EUR`[[2]](#closest-currency-2) | `Euro (€)` | :white_check_mark: |
| `th-th` | `ประเทศไทย` | `USD`[[3]](#closest-currency-3) | `US Dollar ($)` | :white_check_mark: |
| `id-id` | `Bahasa Indonesia` | `IDR` | `Indonesian Rupiah (Rp)` | :white_check_mark: |
| `it-it` | `Italiano` | `EUR` | `Euro (€)` | :white_check_mark: |
| | | `CHF`[[9]](#closest-culture-9) | `Swiss Franc. (chf)` | :white_check_mark: |
| `hu-hu` | `Magyar` | `EUR`[[1]](#closest-currency-1) | `Euro (€)` | :white_check_mark: |
| `nb-no` | `Norsk` | `NOK` | `Norwegian Krone (kr)` | :white_check_mark: |
| `nl-nl` | `Nederlands` | `EUR` | `Euro (€)` | :white_check_mark: |
| `pl-pl` | `Polski` | `EUR`[[1]](#closest-currency-1) | `Euro (€)` | :white_check_mark: |
| `pt-br` | `Português (Brasil)` | `BRL` | `Brazilian Real (R$)` | :white_check_mark: |
| `pt-pt` | `Português` | `EUR` | `Euro (€)` | :white_check_mark: |
| `sv-se` | `Svenska` | `SEK` | `Swedish Krona (kr)` | :white_check_mark: |
| `tr-tr` | `Türkçe` | `TRY` | `Turkish Lira (TL)` | :white_check_mark: |
| `ru-ru` | `Pусский` | `RUB` | `Russian Ruble (руб)` | :white_check_mark: |
| `ja-jp` | `日本語` | `JPY` | `Japanese Yen (¥)` | :white_check_mark: |
| `ko-kr` | `한국어` | `KRW` | `Korean Won (₩)` | :white_check_mark: |
| `zh-cn` | `中文(简体)` | `N/A` | `N/A` | `N/A` |
| `zh-tw` | `中文(繁體)` | `TWD` | `Taiwanese Dollar (NT$)` | :x: |
| | | `HKD`[[10]](#closest-culture-10) | `Hong Kong Dollar (HK$)` | :x: |

<a id="closest-currency-1">1.</a> Euro is used for countries which don't have their currency listed, are [part of the European Union but not part of the Eurozone][european-union].

<a id="closest-currency-2">2.</a> Euro is used for Iceland because its [biggest trading partners][iceland-import-export] are using it.

<a id="closest-currency-3">3.</a> USD is used when no other currency could be matched to the country.

<a id="closest-culture-4">4.</a> English (UK) has been selected due to the use of [South African English][south-african-english] in South Africa.

<a id="closest-culture-5">5.</a> Spanish is considered to be the closest language to [Rioplatense Spanish][rioplatense-spanish]

<a id="closest-culture-6">6.</a> English (UK) has been selected due to the use of [Malaysian English][malaysian-english] in Malaysia.

<a id="closest-culture-7">7.</a> English (UK) has been selected due to the use of [New Zealand English][new-zealand-english] in New Zealand.

<a id="closest-currency-8">8.</a> USD is used because the Saudi riyal is [pegged with][saudi-riyal-fixed-exchange-rate] the US Dollar.

<a id="closest-culture-9">9.</a> German, French and Italian are three of the [official languages][swizerland-official-languages] of Switzerland.

<a id="closest-culture-10">10.</a> English is one of the [official languages][hong-kong-traditional-chinese-english] of Hong-Kong. Traditional Chinese is one of the [official scripts][hong-kong-traditional-chinese-english] of Hong Kong, `zh-tw` is the only other culture available using Traditional Chinese.

:rotating_light: the parser is not - yet - able to retrieve pricing for the regions `east-china2`, `north-china2`, `east-china` and `north-china` as it is available on a [different website][azure-china].

:rotating_light: the parser is not able to retrieve pricing for the regions `us-dod-central` and `us-dod-east` as no virtual machines are listed as publicly available.

### Parser pre-requisites

- `Node.js 8.12`
- `Yarn 1.13.0`

```posh
λ  cd .\parser\
λ  yarn
```

### Parser usage

```posh
λ  cd .\parser\
λ  yarn crawl --culture en-us --currency USD --operating-system linux --region us-west
```

You can also use short names:

```posh
λ  yarn crawl -l en-us -c USD -o linux -r us-west
```

Arguments:

- `culture` any of the `option` `value` in the **Culture** `select`
  - **This will impact the formatting of the pricing**
- `currency` any of the `option` `value` in the **Currency** `select`
- `operating-system` any of the `option` `value` in the **OS/Software** `select`
- `region` any of the `option` `value` in the **Region** `select`

![OS and Region select](docs/assets/os-region.png)

In the footer:

![Culture and Currency select](docs/assets/culture-currency.png)

### Parser output

Writes `2` output files in the `out\` directory. One is a `CSV`, the other one is `JSON`. Both files contain the same data.

```text
.\out\vm-pricing_<region>_<operating-system>.csv
.\out\vm-pricing_<region>_<operating-system>.json
```

Fields:

- `Instance`
- `vCPU`
- `RAM`
- `Pay as You Go`
- `One Year Reserved`
- `Three Year Reserved`
- `Three Year Reserved With Azure Hybrid Benefit`

## Coster

Price `VMs` using the `JSON` pricing files generated by the `Parser`. The `Coster` will select the cheapest `VM` that has enough `CPU` cores and `RAM`.

### Coster pre-requisites

- `.NET Core SDK 2.2`

### Coster usage

You should paste the `JSON` pricing files generated by the `Parser` in the `Pricing\` folder.

In `Release` mode:

```posh
λ  cd .\coster\src\AzureVmCoster
λ  dotnet run --configuration Release -- <path>
```

In `Debug` mode

```posh
λ  cd .\coster\src\AzureVmCoster
λ  dotnet run --configuration Debug
Input file path:
```

You'll need to provide the `<path>` when prompted.

`<path>` should point to a `CSV` file with the following fields:

- `Region`
- `Name`
- `CPU` (`short`)
- `RAM` (in `GB`, a `decimal`)
- `Operating System`

The columns can be in any order and the `CSV` file can contain extra-columns. The `Region` and `Operating System` fields must match existing regions and supported operating systems in `Azure`.

### Coster output

The `Coster` will generate a `CSV` file in the `Out\` directory with the following fields:

- `Region`
- `Name`
- `Operating System`
- `Instance`
- `CPU`
- `RAM`
- `Pay as You Go`
- `One Year Reserved`
- `Three Year Reserved`
- `Three Year Reverved with Azure Hybrid Benefit`

[virtual-machines-pricing]: https://azure.microsoft.com/en-au/pricing/details/virtual-machines/windows/
[managed-disks-pricing]: https://azure.microsoft.com/en-us/pricing/details/managed-disks/
[bandwidth-pricing-details]: https://azure.microsoft.com/en-us/pricing/details/bandwidth/
[iceland-import-export]: https://atlas.media.mit.edu/en/profile/country/isl/#Destinations
[south-african-english]: https://en.wikipedia.org/wiki/South_African_English
[rioplatense-spanish]: https://en.wikipedia.org/wiki/Rioplatense_Spanish
[malaysian-english]: https://en.wikipedia.org/wiki/Malaysian_English
[new-zealand-english]: https://en.wikipedia.org/wiki/New_Zealand_English
[saudi-riyal-fixed-exchange-rate]: https://en.wikipedia.org/wiki/Saudi_riyal#Fixed_exchange_rate
[european-union]: https://europa.eu/european-union/about-eu/countries_en#tab-0-0
[swizerland-official-languages]: https://en.wikipedia.org/wiki/Switzerland#Languages
[azure-china]: https://www.azure.cn/en-us/pricing/details/virtual-machines/
[hong-kong-traditional-chinese-english]: https://en.wikipedia.org/wiki/Hong_Kong#cite_note-language-status-8
