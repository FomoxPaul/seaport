![Seaport](img/Seaport-banner.png)
![海港](img/Seaport-banner.png)


# Seaport
# 海港


Seaport is a new marketplace protocol for safely and efficiently buying and selling NFTs.
Seaport 是一种新的市场协议，用于安全有效地买卖 NFT。


## Table of Contents
## 目录


- [Background](#background)
- [背景]（#背景）
- [Deployments](#deployments)
- [部署](#部署)
- [Install](#install)
- [安装](#安装)
- [Usage](#usage)
- [用法](#用法)
- [Audits](#audits)
- [审计](#审计)
- [Contributing](#contributing)
- [贡献](#贡献)
- [License](#license)
- [许可证](#许可证)


## Background
## 背景


Seaport is a marketplace protocol for safely and efficiently buying and selling NFTs. Each listing contains an arbitrary number of items that the offerer is willing to give (the "offer") along with an arbitrary number of items that must be received along with their respective receivers (the "consideration").
Seaport 是一种用于安全有效地买卖 NFT 的市场协议。每个列表包含任意数量的要约人愿意提供的物品（“要约”），以及必须连同其各自的接收者一起接收的任意数量的物品（“对价”）。


See the [documentation](docs/SeaportDocumentation.md), the [interface](contracts/interfaces/SeaportInterface.sol), and the full [interface documentation](https://docs.opensea.io/v2.0/reference/seaport-overview) for more information on Seaport.
请参阅[文档](docs/SeaportDocumentation.md)、[接口](contracts/interfaces/SeaportInterface.sol) 和完整的[接口文档](https://docs.opensea.io/v2.0/reference /seaport-overview) 了解有关海港的更多信息。


## Deployments
## 部署


Seaport deployment addresses:
海港部署地址：


|网络 |地址 |
| ---------------- | ------------------------------------------ |
| Ethereum Mainnet | [0x00000000006CEE72100D161c57ADA5Bb2be1CA79](https://etherscan.io/address/0x00000000006cee72100d161c57ada5bb2be1ca79#code) |
| Polygon Mainnet  | [0x00000000006CEE72100D161c57ADA5Bb2be1CA79](https://polygonscan.com/address/0x00000000006CEE72100D161c57ADA5Bb2be1CA79) |
| Goerli           | [0x00000000006CEE72100D161c57ADA5Bb2be1CA79](https://goerli.etherscan.io/address/0x00000000006cee72100d161c57ada5bb2be1ca79#code) |
| Rinkeby          | [0x00000000006CEE72100D161c57ADA5Bb2be1CA79](https://rinkeby.etherscan.io/address/0x00000000006cee72100d161c57ada5bb2be1ca79#code) |


Conduit Controller deployment addresses:
导管控制器部署地址：


|网络 |地址 |
| ---------------- | ------------------------------------------ |
| Ethereum Mainnet | [0x00000000006cE100a8b5eD8eDf18ceeF9e500697](https://etherscan.io/address/0x00000000006ce100a8b5ed8edf18ceef9e500697#code) |
| Polygon Mainnet  | [0x00000000006cE100a8b5eD8eDf18ceeF9e500697](https://polygonscan.com/address/0x00000000006ce100a8b5ed8edf18ceef9e500697) |
| Goerli           | [0x00000000006cE100a8b5eD8eDf18ceeF9e500697](https://goerli.etherscan.io/address/0x00000000006ce100a8b5ed8edf18ceef9e500697) |
| Rinkeby          | [0x00000000006cE100a8b5eD8eDf18ceeF9e500697](https://rinkeby.etherscan.io/address/0x00000000006ce100a8b5ed8edf18ceef9e500697) |


## Install
## 安装


To install dependencies and compile contracts:
要安装依赖项并编译合约：


```bash
git clone https://github.com/ProjectOpenSea/seaport && cd seaport
yarn install
yarn build
```


## Usage
## 用法


To run hardhat tests written in javascript:
要运行用 javascript 编写的安全帽测试：


```bash
yarn test
yarn coverage
```


> Note: artifacts and cache folders may occasionally need to be removed between standard and coverage test runs.
> 注意：有时可能需要在标准和覆盖测试运行之间删除工件和缓存文件夹。


To run hardhat tests against reference contracts:
要针对参考合约运行安全帽测试：


```bash
yarn test:ref
yarn coverage:ref
```


To profile gas usage:
要分析气体使用情况：


```bash
yarn profile
```


### Foundry Tests
### 铸造测试


Seaport also includes a suite of fuzzing tests written in solidity with Foundry.
Seaport 还包括一套适用 Foundry 一起编写的测试用例。


To install Foundry (assuming a Linux or macOS system):
要安装 Foundry（适用于Linux 或 macOS 系统）：


```bash
curl -L https://foundry.paradigm.xyz | bash
```


This will download foundryup. To start Foundry, run:
需先下载foundryup。然后启动 Foundry，请运行：


```bash
foundryup
```


To install dependencies:
要安装依赖项：

```
forge install
```

To precompile contracts:
预编译合约：

The optimized contracts are compiled using the IR pipeline, which can take a long time to compile. By default, the differential test suite depends deploys precompiled versions of both the optimized and reference contracts. Precompilation can be done by specifying specific Foundry profiles.
优化后的合约是使用 IR 管道编译的，这可能需要很长时间才能编译。默认情况下，差分测试套件依赖于部署优化合约和参考合约的预编译版本。可以通过指定特定的 Foundry 配置文件来完成预编译。

```bash
FOUNDRY_PROFILE=optimized forge build
FOUNDRY_PROFILE=reference forge build
```

There are three Foundry profiles for running the test suites, which bypass the IR pipeline to speed up compilation. To run tests, run any of the following:
有三个 Foundry 配置文件用于运行测试套件，它们绕过 IR 管道以加快编译速度。要运行测试，请运行以下任一命令：


```bash
FOUNDRY_PROFILE=test forge test # with 5000 fuzz runs
FOUNDRY_PROFILE=lite forge test # 1000 次模糊测试
FOUNDRY_PROFILE=local-ffi forge test # 正常编译部署ReferenceConsideration，运行1000次fuzz
```

You may wish to include a `.env` file that `export`s a specific profile when developing locally.
您可能希望在本地开发时包含一个“导出”特定配置文件的“.env”文件。

**Note** that stack+debug traces will not be available for precompiled contracts. To facilitate local development, specifying `FOUNDRY_PROFILE=local-ffi` will compile and deploy the reference implementation normally, allowing for stack+debug traces.
**注意** 堆栈+调试跟踪将不适用于预编译合约。为方便本地开发，指定 `FOUNDRY_PROFILE=local-ffi` 将正常编译和部署参考实现，允许堆栈+调试跟踪。

**Note** the `local-ffi` profile uses Forge's `ffi` flag. `ffi` can potentially be unsafe, as it allows Forge to execute arbitrary code. Use with caution, and always ensure you trust the code in this repository, especially when working on third-party forks.
**注意** `local-ffi` 配置文件使用 Forge 的 `ffi` 标志。 `ffi` 可能是不安全的，因为它允许 Forge 执行任意代码。谨慎使用，并始终确保您信任此存储库中的代码，尤其是在使用第三方分叉时。

The following modifiers are also available:
以下修饰符也可用：

- Level 2 (-vv): Logs emitted during tests are also displayed.
- 级别 2 (-vv)：还显示测试期间发出的日志。
- Level 3 (-vvv): Stack traces for failing tests are also displayed.
- 级别 3 (-vvv)：还显示失败测试的堆栈跟踪。
- Level 4 (-vvvv): Stack traces for all tests are displayed, and setup traces for failing tests are displayed.
- 级别 4 (-vvvv)：显示所有测试的堆栈跟踪，并显示失败测试的设置跟踪。
- Level 5 (-vvvvv): Stack traces and setup traces are always displayed.
- 级别 5 (-vvvvv)：始终显示堆栈跟踪和设置跟踪。

```bash
FOUNDRY_PROFILE=test forge test  -vv
```


For more information on foundry testing and use, see [Foundry Book installation instructions](https://book.getfoundry.sh/getting-started/installation.html).
有关 Foundry 测试和使用的更多信息，请参阅 [Foundry Book 安装说明](https://book.getfoundry.sh/getting-started/installation.html)。


To run lint checks:
要运行 lint 检查：


```bash
yarn lint:check
```


Lint checks utilize prettier, prettier-plugin-solidity, and solhint.
Lint 检查使用 prettier、prettier-plugin-solidity 和 solhint。


```javascript
"prettier": "^2.5.1",
"prettier-plugin-solidity": "^1.0.0-beta.19"
```

## Audits
## 审计


OpenSea engaged Trail of Bits to audit the security of Seaport. From April 18th to May 12th 2022, a team of Trail of Bits consultants conducted a security review of Seaport. The audit did not uncover significant flaws that could result in the compromise of a smart contract, loss of funds, or unexpected behavior in the target system. Their [full report is available here](https://github.com/trailofbits/publications/blob/master/reviews/SeaportProtocol.pdf).
OpenSea 聘请 Trail of Bits 来审计 Seaport 的安全性。从 2022 年 4 月 18 日到 5 月 12 日，Trail of Bits 顾问团队对 Seaport 进行了安全审查。审计没有发现可能导致智能合约受损、资金损失或目标系统出现意外行为的重大缺陷。他们的 [完整报告可在此处获得](https://github.com/trailofbits/publications/blob/master/reviews/SeaportProtocol.pdf)。


## Contributing
## 贡献


Contributions to Seaport are welcome by anyone interested in writing more tests, improving readability, optimizing for gas efficiency, or extending the protocol via new zone contracts or other features.
任何有兴趣编写更多测试、提高可读性、优化 gas 效率或通过新区域合约或其他功能扩展协议的人都欢迎对 Seaport 的贡献。


When making a pull request, ensure that:
发出拉取请求时，请确保：


- All tests pass.
- 所有测试通过。
- Code coverage remains at 100% (coverage tests must currently be written in hardhat).
- 代码覆盖率保持在 100%（覆盖率测试当前必须用安全帽编写）。
- All new code adheres to the style guide:
- 所有新代码都遵循样式指南：
- All lint checks pass.
- 所有皮棉检查通过。
- Code is thoroughly commented with natspec where relevant.
- 代码在相关的地方用 natspec 进行了彻底的注释。
- If making a change to the contracts:
- 如果对合同进行更改：
- Gas snapshots are provided and demonstrate an improvement (or an acceptable deficit given other improvements).
- 提供气体快照并展示改进（或在其他改进情况下可接受的不足）。
- Reference contracts are modified correspondingly if relevant.
- 如果相关，参考合同会相应修改。
- New tests (ideally via foundry) are included for all new features or code paths.
- 所有新功能或代码路径都包含新测试（最好通过代工厂）。
- If making a modification to third-party dependencies, `yarn audit` passes.
- 如果对第三方依赖项进行修改，`yarn audit` 通过。
- A descriptive summary of the PR has been provided.
- 提供了 PR 的描述性摘要。


## License
## 许可证


[MIT](LICENSE) Copyright 2022 Ozone Networks, Inc.
[MIT]（许可证）版权所有 2022 Ozone Networks, Inc.
