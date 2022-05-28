# Code4rena Guidelines
# Code4rena 指南


## Overview:
## 概述：


Seaport is a marketplace protocol for safely and efficiently buying and selling NFTs. Each listing contains an arbitrary number of items that the offerer is willing to give (the "offer") along with an arbitrary number of items that must be received along with their respective receivers (the "consideration").
Seaport 是一种用于安全有效地买卖 NFT 的市场协议。每个上架包含任意数量的要约人愿意提供的物品（“要约”），以及必须连同其各自的接收者一起接收的任意数量的物品（“对价”）。


## In Scope
## 适用范围


At a high level, the core invariants that we expect to be upheld are that:
在高层次上，我们期望得到维护的核心不变内容如下：


- Only items that are explicitly offered as part of a valid order may be transferred from an offerer’s account as long as they only set token approvals on either Seaport directly or on a conduit that only has Seaport set as a channel.
- 只有作为有效订单的一部分明确提供的项目才可以从提供者的帐户中转移，只要他们只在 Seaport 直接或仅将 Seaport 设置为渠道的管道上设置令牌批准。
- No order fulfillment may spend more than the offer items that are explicitly set for the order in question. Note that not all offer items need to be spent.
- 任何订单履行的花费不得超过为相关订单明确设置的优惠项目。请注意，并非所有优惠项目都需要花费。
- All consideration items (or fractions thereof in the case of orders that support partial fills) must be received in full by the named recipients in order for the corresponding offer items (or fractions thereof) to be spent. Note that additional consideration items may be added to any order on fulfillment as “tips”. Note also that when calling any fulfillment method other than `matchOrders` or `matchAdvancedOrders` that an implied “mirror” order is created for the fulfiller and so offer items on fulfilled orders should be treated as consideration items for the fulfiller (with the exception of `fulfillBasicOrder` where ERC721 ⇒ ERC20 and ERC1155 ⇒ ERC20 route types will use a portion of the offered item on the fulfilled order to pay out consideration items on that order).
- 所有对价项目（或支持部分成交的订单的部分）必须由指定的接收者全额接收，以便花费相应的报价项目（或部分）。请注意，额外的考虑项目可能会作为“提示”添加到任何订单中。另请注意，当调用除 `matchOrders` 或 `matchAdvancedOrders` 之外的任何履行方法时，会为履行者创建隐含的“镜像”订单，因此履行订单上的商品应视为履行者的考虑项目（除了`fulfillBasicOrder` 其中 ERC721 ⇒ ERC20 和 ERC1155 ⇒ ERC20 路线类型将使用已履行订单上提供的商品的一部分来支付该订单上的对价商品）。
- In all cases, assume that that all items contain standard ERC20/721/1155 behavior. This may include popular tokens or contracts (though reporting particular tokens that would violate these invariants would be categorized as a low-severity finding).
- 在所有情况下，假设所有项目都包含标准 ERC20/721/1155 行为。这可能包括流行的代币或合约（尽管报告违反这些不变量的特定代币将被归类为低严重性发现）。


## Out of scope
## 超出范围


There are a number of known limitations that are explicitly out of scope for the context of the competition:
有许多已知的限制明确超出了适用的范围：


- As a malicious or vulnerable conduit owner may set a channel on a conduit that allows for approved tokens to be taken at will, we make the assumption in the context of this contest that only the Seaport contract will be added as a channel to any conduit.
- 由于恶意或易受攻击的管道所有者可能会在管道上设置允许随意获取已批准代币的通道，因此我们在本次竞赛的背景下假设只有 Seaport 合约将作为通道添加到任何管道.
- As all offer and consideration items are allocated against one another in memory, there are scenarios in which the actual received item amount will differ from the amount specified by the order — notably, this includes items with a fee-on-transfer mechanic.
- 由于所有报价和对价项目都在内存中相互分配，因此在某些情况下，实际收到的项目数量将与订单指定的数量不同 - 特别是，这包括具有转移费用机制的项目。
- As all offer items are taken directly from the offerer and all consideration items are given directly to the named recipient, there are scenarios where those accounts can increase the gas cost of order fulfillment or block orders from being fulfilled outright depending on the item being transferred. If the item in question is Ether or a similar native token, a recipient can throw in the payable fallback or even spend excess gas from the submitter. Similar mechanics can be leveraged by both offerers and receives if the item in question is a token with a transfer hook (like ERC1155 and ERC777) or a non-standard token implementation.
- 由于所有报价项目都直接从报价方获取，所有对价项目都直接提供给指定的收件人，因此在某些情况下，这些帐户可能会增加订单履行的 gas 成本或阻止订单完全履行，具体取决于被转移的项目.如果有问题的项目是以太币或类似的原生代币，接收者可以投入应付的后备，甚至从提交者那里花费多余的气体。如果所讨论的项目是带有传输挂钩的代币（如 ERC1155 和 ERC777）或非标准代币实现，则提供者和接收者都可以利用类似的机制。
- As fulfillments may be executed in whatever sequence the fulfiller specifies as long as the fulfillments are all executable, as restricted orders are validated via zones prior to execution, and as orders may be combined with other orders or have additional consideration items supplied, **any items with modifiable state are at risk of having that state modified during execution** if a payable Ether recipient or onReceived 1155 transfer hook is able to modify that state. By way of example, imagine an offerer offers WETH and requires some ERC721 item as consideration, where the ERC721 should have some additional property like not having been used to mint some other ERC721 item. Then, even if the offerer enforces that the ERC721 have that property via a restricted order that checks for the property, a malicious fulfiller could include a second order (or even just an additional consideration item) that uses the ERC721 item being sold to mint before it is transferred to the offerer.
- 因为履行可以按照履行者指定的任何顺序执行，只要履行都是可执行的，因为受限订单在执行之前通过区域验证，并且订单可以与其他订单合并或提供额外的考虑项目，**如果应付以太币接收者或 onReceived 1155 转移挂钩能够修改该状态，则任何具有可修改状态的项目都存在在执行期间修改该状态的风险**。举例来说，假设提供者提供 WETH 并需要一些 ERC721 项目作为考虑，其中 ERC721 应该具有一些额外的属性，例如没有用于铸造一些其他 ERC721 项目。然后，即使报价方通过检查财产的受限订单强制 ERC721 拥有该财产，恶意履行者也可能包括使用之前出售给铸币厂的 ERC721 项目的第二个订单（甚至只是一个额外的考虑项目）它被转移给要约人。
- As all consideration items are supplied at the time of order creation, dynamic adjustment of recipients or amounts after creation (e.g. modifications to royalty payout info) is not supported.
- 由于所有对价项目均在创建订单时提供，因此不支持在创建后动态调整收件人或金额（例如修改版税支付信息）。
- As all criteria-based items are tied to a particular token, there is no native way to construct orders where items specify cross-token criteria. Additionally, each potential identifier for a particular criteria-based item must have the same amount as any other identifier.
- 由于所有基于标准的项目都与特定令牌相关联，因此没有本地方式来构建项目指定跨令牌标准的订单。此外，基于特定标准的项目的每个潜在标识符必须与任何其他标识符具有相同的数量。
- As orders that contain items with ascending or descending amounts may not be filled as quickly as a fulfiller would like (e.g. transactions taking longer than expected to be included), there is a risk that fulfillment on those orders will supply a larger item amount, or receive back a smaller item amount, than they intended or expected.
- 由于包含金额递增或递减的项目的订单可能不会像履行者希望的那样快速履行（例如，交易花费的时间比预期的要长），这些订单的履行可能会提供更大的项目数量，或收到比他们预期或预期的更少的物品数量。
- As all items on orders supporting partial fills must be "cleanly divisible" when performing a partial fill, orders with multiple items should to be constructed with care. A straightforward heuristic is to start with a "unit" bundle (e.g. 1 NFT item A, 3 NFT item B, and 5 NFT item C for 2 ETH) then applying a multiple to that unit bundle (e.g. 7 of those units results in a partial order for 7 NFT item A, 21 NFT item B, and 35 NFT item C for 14 ETH).
- 由于支持部分成交的订单上的所有项目在执行部分成交时必须是“完全可分割的”，因此应谨慎构建具有多个项目的订单。一个简单的启发式方法是从一个“单元”捆绑包开始（例如 1 个 NFT 项目 A、3 个 NFT 项目 B 和 5 个 NFT 项目 C 用于 2 ETH），然后将倍数应用于该单元捆绑包（例如，这些单元中的 7 个导致7 NFT 项目 A、21 NFT 项目 B 和 35 NFT 项目 C 的部分订单（14 ETH）。
- As Ether cannot be "taken" from an account, any order that contains Ether or other native tokens as an offer item (including "implied" mirror orders) must be supplied by the caller executing the order(s) as msg.value. This also explains why there are no `ERC721_TO_ERC20` and `ERC1155_TO_ERC20` basic order route types, as Ether cannot be taken from the offerer in these cases. One important takeaway from this mechanic is that, technically, anyone can supply Ether on behalf of a given offerer (whereas the offerer themselves must supply all other items). It also means that all Ether must be supplied at the time the order or group of orders is originally called (and the amount available to spend by offer items cannot be increased by an external source during execution as is the case for token balances).
- 由于不能从账户中“获取”以太币，任何包含以太币或其他原生代币作为报价项目的订单（包括“隐含”镜像订单）必须由执行订单的调用者作为 msg.value 提供。这也解释了为什么没有 `ERC721_TO_ERC20` 和 `ERC1155_TO_ERC20` 基本订单路由类型，因为在这些情况下无法从提供者那里获取以太币。这个机制的一个重要收获是，从技术上讲，任何人都可以代表给定的提议者提供以太币（而提议者自己必须提供所有其他物品）。这也意味着必须在最初调用订单或订单组时提供所有以太币（并且在执行期间，外部来源不能增加报价项目可花费的金额，就像代币余额的情况一样）。
- As extensions to the consideration array on fulfillment (i.e. "tipping") can be arbitrarily set by the caller, fulfillments where all matched orders have already been signed for or validated can be frontrun on submission, with the frontrunner modifying any tips. Therefore, it is important that orders fulfilled in this manner either leverage "restricted" order types with a zone that enforces appropriate allocation of consideration extensions, or that each offer item is fully spent and each consideration item is appropriately declared on order creation.
- 由于调用者可以任意设置履行时考虑数组的扩展（即“小费”），所有匹配订单已经签署或验证的履行可以在提交时抢先运行，领先者修改任何提示。因此，重要的是，以这种方式完成的订单要么利用具有强制适当分配对价扩展的区域的“受限”订单类型，要么每个报价项目都已完全用完，并且每个对价项目都在订单创建时适当声明。
- As orders that have been verified (via a call to `validate`) or partially filled will skip signature validation on subsequent fulfillments, orders that utilize EIP-1271 for verifying orders may end up in an inconsistent state where the original signature is no longer valid but the order is still fulfillable. In these cases, the offerer must explicitly cancel the previously verified order in question if they no longer wish for the order to be fulfillable.
- 由于已验证（通过调用 `validate`）或部分履行的订单将在后续履行时跳过签名验证，使用 EIP-1271 验证订单的订单最终可能会处于原始签名不再存在的不一致状态有效，但订单仍可履行。在这些情况下，如果供应商不再希望订单可以履行，他们必须明确取消先前已验证的相关订单。
- As orders filled by the "fulfill available" method will only be skipped if those orders have been cancelled, fully filled, or are inactive, fulfillments may still be attempted on unfulfillable orders (examples include revoked approvals or insufficient balances). This scenario (as well as issues with order formatting) will result in the full batch failing.
- 由于通过“可履行”方法履行的订单仅在这些订单已被取消、已完全履行或处于非活动状态时才会被跳过，因此仍可能对无法履行的订单尝试履行（例如，撤销批准或余额不足）。这种情况（以及订单格式问题）将导致整批失败。
- As order parameters must be supplied upon cancellation, orders that were meant to remain private (e.g. were not published publicly) will be made visible upon cancellation. While these orders would not be *fulfillable* without a corresponding signature, cancellation of private orders without broadcasting intent currently requires the offerer (or the zone, if the order type is restricted and the zone supports it) to increment the nonce.
- 由于必须在取消时提供订单参数，因此原本是保密的（例如未公开发布的）订单将在取消时可见。虽然这些订单如果没有相应的签名就无法*履行*，但在没有广播意图的情况下取消私人订单目前需要提供者（或区域，如果订单类型受到限制并且区域支持它）增加随机数。
- As order fulfillment attempts may become public before being included in a block, there is a risk of those orders being front-run. This risk is magnified in cases where offered items contain ascending amounts or consideration items contain descending amounts, as there is added incentive to leave the order unfulfilled until another interested fulfiller attempts to fulfill the order in question.
- 由于订单履行尝试可能会在被包含在区块中之前公开，因此这些订单存在被抢先交易的风险。如果提供的商品包含递增金额或对价商品包含递减金额，则这种风险会被放大，因为在另一个感兴趣的履行者尝试履行相关订单之前，会有额外的动机让订单未履行。
- As validated orders may still be unfulfillable due to invalid item amounts or other factors, callers should determine whether validated orders are fulfillable by simulating the fulfillment call prior to execution. Also note that anyone can validate a signed order, but only the offerer can validate an order without supplying a signature.
- 由于无效的商品数量或其他因素，已验证的订单可能仍无法履行，调用方应在执行前通过模拟履行调用来确定已验证的订单是否可以履行。另请注意，任何人都可以验证已签名的订单，但只有提供者可以在不提供签名的情况下验证订单。
- As the offerer or the zone of a given order may cancel an order that differs from the intended order, callers should ensure that the intended order was cancelled by calling `getOrderStatus` and confirming that `isCancelled` returns `true`.
- 由于给定订单的提供者或区域可能会取消与预期订单不同的订单，调用者应通过调用 `getOrderStatus` 并确认 `isCancelled` 返回 `true` 来确保预期订单已取消。
- As all derived amounts of partial fills and ascending/descending orders need to be derived without integer overflows, some categories of order may end up in a state where they can no longer be fulfilled due to reverting amount calculations.
- 由于部分执行和升/降订单的所有派生金额都需要在没有整数溢出的情况下派生，因此某些类别的订单可能会由于还原金额计算而最终处于无法再履行的状态。
- As many functions expect the default ABI encoding to be used, calling functions with non-standard encoding should not be expected to succeed.
- 由于许多函数都希望使用默认的 ABI 编码，因此不应期望使用非标准编码调用函数会成功。
- As ERC1271-compliant wallets implement their own signature verification, there is a risk that an improperly configured ERC1271 offerer could have funds stolen due to overly permissive signature verification.
- 由于符合 ERC1271 的钱包实施了自己的签名验证，因此存在配置不当的 ERC1271 提供者可能因签名验证过于宽松而导致资金被盗的风险。
- More generally, any finding reported in the Trail of Bits audit is additionally out of scope.
- 更一般地说，Trail of Bits 审计中报告的任何发现都超出了范围。


## Tiers:
## 层级：


**Low / Informational:** 
**低/信息：**


- Informational issues, like returning the wrong error type.
- 信息性问题，例如返回错误的错误类型。
- Issues with the reference implementation where behavior does not map 1:1 with the optimized contracts (with the exception of revert reasons as some are not reproducible without optimizations)
- 参考实现的问题，其中行为未与优化的合约 1:1 映射（除了还原原因，因为有些在没有优化的情况下无法重现）
- Gas optimizations.
- 气体优化。


**Medium:**
**中等的：**


- Any behavior that is not in line with expected behavior on standard interaction with the protocol (bearing in mind all known limitations) — this would include a halt of functionality where orders that should succeed revert, or edge cases that do not result in widespread loss of funds but might lead to a small subset of funds being at risk.
- 在与协议的标准交互中与预期行为不符的任何行为（记住所有已知限制）——这将包括停止功能，其中应成功恢复的订单，或不会导致广泛损失的边缘情况资金，但可能导致一小部分资金面临风险。


**High / Critical**
**高/严重**


- Any of the core invariants listed above being exploitable in a manner that places most or all user funds at risk.
- 上面列出的任何核心不变量都可以通过将大部分或所有用户资金置于风险中的方式加以利用。


### **Areas of focus**
### **重点领域**


While wardens should submit any bugs they identify for review, we particularly encourage review of code which has any of the following:
虽然管理员应提交他们发现的任何错误以供审查，但我们特别鼓励审查具有以下任何一项的代码：


- transfer of multiple assets
- 转移多项资产
- arithmetic for order amounts
- 订单金额的计算
- aggregation of fulfillments
- 履行的聚合
  - FulfillmentApplier.sol

- transfer accumulation
- 转移积累
  - Executor.sol
  - OrderCombiner.sol
  - OrderFulfiller.sol
  - BasicOrderFulfiller.sol
- low-level handling of nested dynamic types in calldata or loaded from calldata
- calldata 中嵌套动态类型或从 calldata 加载的低级处理
  - OrderFulfiller.sol
  - BasicOrderFulfiller.sol
  - FulfillmentApplier.sol
  - GettersAndDerivers.sol
  - CriteriaResolution.sol
- order matching / fulfillment validation
- 订单匹配/履行验证
  - OrderFulfiller.sol
  - BasicOrderFulfiller.sol
  - OrderCombiner.sol
  - FulfillmentApplier.sol
  - CriteriaResolution.sol


## **Tests**
## **测试**


A full suite of unit tests using Hardhat and Foundry have been provided in this repo, found in the `test` folder.
此 repo 中提供了使用 Hardhat 和 Foundry 的全套单元测试，可在“test”文件夹中找到。


## Information:
## 信息：


[https://docs.opensea.io/v2.0/reference/seaport-overview](https://docs.opensea.io/v2.0/reference/seaport-overview)


### Reference Implementation:
### 参考实现：


The reference folder has its own implementation of Seaport which is designed to be readable and have feature parity with the Seaport.sol. We created the Reference implementation because a lot of Seaport is optimized by using assembly and interesting memory management techniques, that often make the code hard to read and understand. The Reference should be easy to read and work the same exact way, but it is NOT what is deployed. So if you find an issue with parity or a bug / vulnerability in the reference implementation, please report it but be advised that it will not classify as a medium or high-severity finding.
参考文件夹有它自己的 Seaport 实现，它被设计为可读并且与 Seaport.sol 具有相同的功能。我们创建了参考实现，因为很多 Seaport 都通过使用汇编和有趣的内存管理技术进行了优化，这通常使代码难以阅读和理解。参考应该易于阅读并以完全相同的方式工作，但它不是部署的内容。因此，如果您在参考实现中发现奇偶校验问题或错误/漏洞，请报告它，但请注意它不会归类为中等或高严重性发现。


## Test contracts
## 测试合约


Test contracts and non-solidity files are explicitly out of scope for the competition, though issues and PRs with any new tests you write as part of your investigation are greatly appreciated.
测试合约和非 Solidity 文件明确超出了比赛的范围，但非常感谢您作为调查的一部分编写的任何新测试的问题和 PR。
