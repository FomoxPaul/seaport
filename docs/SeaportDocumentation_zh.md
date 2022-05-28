# Seaport Documentation
# Seaport文档


Documentation around creating orders, fulfillment, and interacting with Seaport.
有关创建订单、履行和与 Seaport 交互的文档。


## Table of Contents
## 目录


- [Order](#order)
- [订单](#order)
- [Order Fulfillment](#order-fulfillment)
- [订单履行](#order-fulfillment)
- [Sequence of Events](#sequence-of-events)
- [事件序列](#sequence-of-events)
- [Known Limitations And Workarounds](#known-limitations-and-workarounds)
- [已知限制和解决方法](#known-limitations-and-workarounds)


## Order
## 订单


Each order contains eleven key components:
每个订单包含十一个关键组件：


- The `offerer` of the order supplies all offered items and must either fulfill the order personally (i.e. `msg.sender == offerer`) or approve the order via signature (either standard 65-byte EDCSA, 64-byte EIP-2098, or an EIP-1271 `isValidSignature` check) or by listing the order on-chain (i.e. calling `validate`).
- 订单的“提供者”供应所有提供的物品，并且必须亲自履行订单（即“msg.sender == 提供者”）或通过签名批准订单（标准 65 字节 EDCSA、64 字节 EIP-2098 ，或 EIP-1271 `isValidSignature` 检查）或通过在链上列出订单（即调用 `validate`）。
- The `zone` of the order is an optional secondary account attached to the order with two additional privileges:
- 订单的“区域”是附加到订单的可选二级账户，具有两个额外的权限：
- The zone may cancel orders where it is named as the zone by calling `cancel`. (Note that offerers can also cancel their own orders, either individually or for all orders signed with their current nonce at once by calling `incrementNonce`).
- 区域可以通过调用`cancel`取消被命名为区域的订单。 （请注意，提供者也可以单独取消他们自己的订单，也可以通过调用“incrementNonce”一次性取消所有使用当前 nonce 签名的订单）。
- "Restricted" orders (as specified by the order type) must either be executed by the zone or the offerer, or must be approved as indicated by a call to an `isValidOrder` or `isValidOrderIncludingExtraData` view function on the zone.
- “受限”订单（由订单类型指定）必须由区域或要约人执行，或者必须通过调用区域上的 `isValidOrder` 或 `isValidOrderIncludingExtraData` 视图函数来获得批准。
- The `offer` contains an array of items that may be transferred from the offerer's account, where each item consists of the following components:
- `offer` 包含一系列可以从提供者帐户转移的项目，其中每个项目由以下组件组成：
- The `itemType` designates the type of item, with valid types being Ether (or other native token for the given chain), ERC20, ERC721, ERC1155, ERC721 with "criteria" (explained below), and ERC1155 with criteria.
- `itemType` 指定项目的类型，有效类型为 Ether（或给定链的其他原生令牌）、ERC20、ERC721、ERC1155、带有“标准”的 ERC721（解释如下）和带有标准的 ERC1155。
- The `token` designates the account of the item's token contract (with the null address used for Ether or other native tokens).
- `token` 指定项目的代币合约的账户（空地址用于以太币或其他原生代币）。
- The `identifierOrCriteria` represents either the ERC721 or ERC1155 token identifier or, in the case of a criteria-based item type, a merkle root composed of the valid set of token identifiers for the item. This value will be ignored for Ether and ERC20 item types, and can optionally be zero for criteria-based item types to allow for any identifier.
- `identifierOrCriteria` 表示 ERC721 或 ERC1155 令牌标识符，或者在基于标准的项目类型的情况下，表示由项目的有效令牌标识符集组成的 merkle 根。对于 Ether 和 ERC20 项目类型，此值将被忽略，并且对于基于标准的项目类型可以选择为零以允许任何标识符。
- The `startAmount` represents the amount of the item in question that will be required should the order be fulfilled at the moment the order becomes active.
- `startAmount` 表示如果在订单激活时完成订单，则需要的相关商品数量。
- The `endAmount` represents the amount of the item in question that will be required should the order be fulfilled at the moment the order expires. If this value differs from the item's `startAmount`, the realized amount is calculated linearly based on the time elapsed since the order became active.
- `endAmount` 表示如果在订单到期时完成订单，将需要的相关商品数量。如果此值与项目的 `startAmount` 不同，则实际金额将根据订单激活后经过的时间线性计算。
- The `consideration` contains an array of items that must be received in order to fulfill the order. It contains all of the same components as an offered item, and additionally includes a `recipient` that will receive each item. This array may be extended by the fulfiller on order fulfillment so as to support "tipping" (e.g. relayer or referral payments).
- `consideration` 包含一系列必须接收才能完成订单的商品。它包含与提供的项目相同的所有组件，并且还包括将接收每个项目的“收件人”。该数组可以由履行者在订单履行时扩展，以支持“小费”（例如中继者或推荐支付）。
- The `orderType` designates one of four types for the order depending on two distinct preferences:
- `orderType` 根据两个不同的偏好为订单指定四种类型之一：
- `FULL` indicates that the order does not support partial fills, whereas `PARTIAL` enables filling some fraction of the order, with the important caveat that each item must be cleanly divisible by the supplied fraction (i.e. no remainder after division).
- `FULL` 表示订单不支持部分填充，而 `PARTIAL` 允许填充订单的一部分，重要的警告是每个项目必须被提供的部分完全整除（即除法后没有余数）。
- `OPEN` indicates that the call to execute the order can be submitted by any account, whereas `RESTRICTED` requires that the order either be executed by the offerer or the zone of the order, or that a magic value indicating that the order is approved is returned upon calling an `isValidOrder` or `isValidOrderIncludingExtraData` view function on the zone.
- `OPEN` 表示执行订单的调用可以由任何账户提交，而 `RESTRICTED` 要求订单要么由要约人执行，要么由订单所在的区域执行，或者是一个魔术值，指示订单是在区域上调用 `isValidOrder` 或 `isValidOrderIncludingExtraData` 视图函数时返回批准。
- The `startTime` indicates the block timestamp at which the order becomes active.
- `startTime` 指示订单激活的区块时间戳。
- The `endTime` indicates the block timestamp at which the order expires. This value and the `startTime` are used in conjunction with the `startAmount` and `endAmount` of each item to derive their current amount.
- `endTime` 表示订单到期的区块时间戳。该值和 `startTime` 与每个项目的 `startAmount` 和 `endAmount` 一起使用，以得出它们的当前数量。
- The `zoneHash` represents an arbitrary 32-byte value that will be supplied to the zone when fulfilling restricted orders that the zone can utilize when making a determination on whether to authorize the order.
- `zoneHash` 代表一个任意的 32 字节值，当执行受限订单时，该值将提供给该区域，该区域在确定是否授权订单时可以使用该值。
- The `salt` represents an arbitrary source of entropy for the order.
- `salt` 代表订单的任意熵源。
- The `conduitKey` is a `bytes32` value that indicates what conduit, if any, should be utilized as a source for token approvals when performing transfers. By default (i.e. when `conduitKey` is set to the zero hash), the offerer will grant ERC20, ERC721, and ERC1155 token approvals to Seaport directly so that it can perform any transfers specified by the order during fulfillment. In contrast, an offerer that elects to utilize a conduit will grant token approvals to the conduit contract corresponding to the supplied conduit key, and Seaport will then instruct that conduit to transfer the respective tokens.
- `conduitKey` 是一个 `bytes32` 值，指示在执行转移时应使用哪个管道（如果有）作为令牌批准的来源。默认情况下（即当 `conduitKey` 设置为零散列时），提供者将直接向 Seaport 授予 ERC20、ERC721 和 ERC1155 令牌批准，以便它可以在履行期间执行订单指定的任何转移。相反，选择使用管道的提议者将授予与提供的管道密钥相对应的管道合同的代币批准，然后海港将指示该管道转移相应的代币。
- The `nonce` indicates a value that must match the current nonce for the given offerer.
- `nonce` 表示一个值，该值必须与给定提议者的当前 nonce 匹配。


## Order Fulfillment
## 订单完成度


Orders are fulfilled via one of four methods:
订单通过以下四种方法之一完成：


- Calling one of two "standard" functions, `fulfillOrder` and `fulfillAdvancedOrder`, where a second implied order will be constructed with the caller as the offerer, the consideration of the fulfilled order as the offer, and the offer of the fulfilled order as the consideration (with "advanced" orders containing the fraction that should be filled alongside a set of "criteria resolvers" that designate an identifier and a corresponding inclusion proof for each criteria-based item on the fulfilled order). All offer items will be transferred from the offerer of the order to the fulfiller, then all consideration items will be transferred from the fulfiller to the named recipient.
- 调用两个“标准”函数之一，`fulfillOrder` 和 `fulfillAdvancedOrder`，其中将构造第二个隐含订单，调用者作为要约人，已履行订单的考虑作为要约，已履行订单的要约作为考虑因素（“高级”订单包含应与一组“标准解析器”一起填写的部分，这些“标准解析器”为已履行订单上的每个基于标准的项目指定标识符和相应的包含证明）。所有报价项目将从订单的提供者转移到履行者，然后所有对价项目将从履行者转移到指定的接收者。
- Calling the "basic" function, `fulfillBasicOrder` with one of six basic route types supplied (`ETH_TO_ERC721`, `ETH_TO_ERC1155`, `ERC20_TO_ERC721`, `ERC20_TO_ERC1155`, `ERC721_TO_ERC20`, and `ERC1155_TO_ERC20`) will derive the order to fulfill from a subset of components, assuming the order in question adheres to the following:
- 使用提供的六种基本路线类型之一（`ETH_TO_ERC721`、`ETH_TO_ERC1155`、`ERC20_TO_ERC721`、`ERC20_TO_ERC1155`、`ERC721_TO_ERC20` 和 `ERC1155_TO_ERC20`）调用“基本”函数、`fulfillBasicOrder` 将导出订单假设有问题的顺序符合以下条件，则从组件子集完成：
- The order only contains a single offer item and contains at least one consideration item.
- 订单仅包含一个报价项目，并且包含至少一个考虑项目。
- The order contains exactly one ERC721 or ERC1155 item and that item is not criteria-based.
- 订单仅包含一件 ERC721 或 ERC1155 商品，并且该商品不是基于标准的。
- The offerer of the order is the recipient of the first consideration item.
- 订单的提供者是第一考虑项目的接收者。
- All other items have the same Ether (or other native tokens) or ERC20 item type and token.
- 所有其他物品都具有相同的以太币（或其他原生代币）或 ERC20 物品类型和代币。
- The order does not offer an item with Ether (or other native tokens) as its item type.
- 该订单不提供以以太币（或其他原生代币）作为其商品类型的商品。
- The `startAmount` on each item must match that item's `endAmount` (i.e. items cannot have an ascending/descending amount).
- 每个项目上的 `startAmount` 必须与该项目的 `endAmount` 匹配（即项目不能有升序/降序金额）。
- All "ignored" item fields (i.e. `token` and `identifierOrCriteria` on native items and `identifierOrCriteria` on ERC20 items) are set to the null address or zero.
- 所有“忽略”的项目字段（即原生项目上的 `token` 和 `identifierOrCriteria` 以及 ERC20 项目上的 `identifierOrCriteria`）设置为空地址或零。
- If the order has an ERC721 item, that item has an amount of `1`.
- 如果订单中有 ERC721 商品，则该商品的数量为 `1`。
- If the order has multiple consideration items and all consideration items other than the first consideration item have the same item type as the offered item, the offered item amount is not less than the sum of all consideration item amounts excluding the first consideration item amount.
- 如果订单有多个对价项目，且除第一对价项目外的所有对价项目与报价项目的项目类型相同，报价项目金额不小于除第一对价项目金额外所有对价项目金额的总和。
- Calling one of two "fulfill available" functions, `fulfillAvailableOrders` and `fulfillAvailableAdvancedOrders`, where a group of orders are supplied alongside a group of fulfillments specifying which offer items can be aggregated into distinct transfers and which consideration items can be accordingly aggregated, and where any orders that have been cancelled, have an invalid time, or have already been fully filled will be skipped without causing the rest of the available orders to revert. Additionally, any remaining orders will be skipped once `maximumFulfilled` available orders have been located. Similar to the standard fulfillment method, all offer items will be transferred from the respective offerer to the fulfiller, then all consideration items will be transferred from the fulfiller to the named recipient.
- 调用两个“履行可用”函数之一，`fulfillAvailableOrders` 和 `fulfillAvailableAdvancedOrders`，其中一组订单与一组履行一起提供，指定哪些报价项目可以聚合到不同的转移中，哪些考虑项目可以相应地聚合，如果任何已被取消、无效时间或已被完全执行的订单将被跳过，而不会导致其余可用订单恢复。此外，一旦找到“maximumFulfilled”可用订单，任何剩余订单都将被跳过。与标准履行方法类似，所有报价项目将从各自的报价者转移到履行者，然后所有对价项目将从履行者转移到指定的接收者。
- Calling one of two "match" functions, `matchOrders` and `matchAdvancedOrders`, where a group of explicit orders are supplied alongside a group of fulfillments specifying which offer items to apply to which consideration items (and with the "advanced" case operating in a similar fashion to the standard method, but supporting partial fills via supplied `numerator` and `denominator` fractional values as well as an optional `extraData` argument that will be supplied as part of a call to the `isValidOrderIncludingExtraData` view function on the zone when fulfilling restricted order types). Note that orders fulfilled in this manner do not have an explicit fulfiller; instead, Seaport will simply ensure coincidence of wants across each order.
- 调用两个“匹配”函数中的一个，“matchOrders”和“matchAdvancedOrders”，其中一组明确的订单与一组履行指定哪些报价项目适用于哪些考虑项目（以及“高级”案例操作以与标准方法类似的方式，但通过提供的“分子”和“分母”小数值以及可选的“extraData”参数支持部分填充，该参数将作为调用“isValidOrderIncludingExtraData”视图函数的一部分提供执行受限订单类型时的区域）。请注意，以这种方式履行的订单没有明确的履行者；相反，Seaport 将简单地确保每个订单的需求一致。


While the standard method can technically be used for fulfilling any order, it suffers from key efficiency limitations in certain scenarios:
虽然标准方法在技术上可用于履行任何订单，但在某些情况下存在关键效率限制：


- It requires additional calldata compared to the basic method for simple "hot paths".
- 与简单“热路径”的基本方法相比，它需要额外的调用数据。
- It requires the fulfiller to approve each consideration item, even if the consideration item can be fulfilled using an offer item (as is commonly the case when fulfilling an order that offers ERC20 items for an ERC721 or ERC1155 item and also includes consideration items with the same ERC20 item type for paying fees).
- 它要求履行者批准每个考虑项目，即使考虑项目可以使用报价项目来履行（在履行为 ERC721 或 ERC1155 项目提供 ERC20 项目的订单时通常会出现这种情况，并且还包括具有相同的 ERC20 项目类型用于支付费用）。
- It can result in unnecessary transfers, whereas in the "match" case those transfers can be reduced to a more minimal set.
- 它可能导致不必要的转移，而在“匹配”情况下，这些转移可以减少到更小的集合。


### Balance and Approval Requirements
### 平衡和批准要求


When creating an offer, the following requirements should be checked to ensure that the order will be fulfillable:
创建报价时，应检查以下要求以确保订单可以履行：


- The offerer should have sufficient balance of all offered items.
- 提供者应在所有提供的项目中有足够的余额。
- If the order does not indicate to use a conduit, the offerer should have sufficient approvals set for the Seaport contract for all offered ERC20, ERC721, and ERC1155 items.
- 如果订单未指示使用管道，则要约人应为所有提供的 ERC20、ERC721 和 ERC1155 项目的海港合同设置足够的批准。
- If the order _does_ indicate to use a conduit, the offerer should have sufficient approvals set for the respective conduit contract for all offered ERC20, ERC721 and ERC1155 items.
- 如果订单_does_ 指示使用管道，则要约人应为所有提供的 ERC20、ERC721 和 ERC1155 项目的相应管道合同设置足够的批准。


When fulfilling a _basic_ order, the following requirements need to be checked to ensure that the order will be fulfillable:
履行_basic_订单时，需要检查以下要求以确保订单可以履行：


- The above checks need to be performed to ensure that the offerer still has sufficient balance and approvals.
- 需要执行上述检查以确保要约人仍有足够的余额和批准。
- The fulfiller should have sufficient balance of all consideration items _except for those with an item type that matches the order's offered item type_ — by way of example, if the fulfilled order offers an ERC20 item and requires an ERC721 item to the offerer and the same ERC20 item to another recipient, the fulfiller needs to own the ERC721 item but does not need to own the ERC20 item as it will be sourced from the offerer.
- 履行者应该对所有考虑项目有足够的余额_除了那些项目类型与订单提供的项目类型匹配的项目_-例如，如果履行的订单提供 ERC20 项目并要求提供者提供 ERC721 项目，并且相同ERC20 项目给另一个收件人，履行者需要拥有 ERC721 项目，但不需要拥有 ERC20 项目，因为它将来自提供者。
- If the fulfiller does not elect to utilize a conduit, they need to have sufficient approvals set for the Seaport contract for all ERC20, ERC721, and ERC1155 consideration items on the fulfilled order _except for ERC20 items with an item type that matches the order's offered item type_.
- 如果履行者不选择使用管道，他们需要为已履行订单上的所有 ERC20、ERC721 和 ERC1155 对价项目的海港合同设置足够的批准 _除了项目类型与订单提供的匹配的 ERC20 项目物品种类_。
- If the fulfiller _does_ elect to utilize a conduit, they need to have sufficient approvals set for their respective conduit for all ERC20, ERC721, and ERC1155 consideration items on the fulfilled order _except for ERC20 items with an item type that matches the order's offered item type_.
- 如果履行者_确实_选择使用管道，则他们需要为已履行订单上的所有 ERC20、ERC721 和 ERC1155 考虑项目为各自的管道设置足够的批准 _除了项目类型与订单提供的项目匹配的 ERC20 项目类型_。
- If the fulfilled order specifies Ether (or other native tokens) as consideration items, the fulfiller must be able to supply the sum total of those items as `msg.value`.
- 如果已履行的订单将以太币（或其他原生代币）指定为考虑项目，则履行者必须能够以“msg.value”的形式提供这些项目的总和。


When fulfilling a _standard_ order, the following requirements need to be checked to ensure that the order will be fulfillable:
履行 _standard_ 订单时，需要检查以下要求以确保订单可以履行：


- The above checks need to be performed to ensure that the offerer still has sufficient balance and approvals.
- 需要执行上述检查以确保要约人仍有足够的余额和批准。
- The fulfiller should have sufficient balance of all consideration items _after receiving all offered items_ — by way of example, if the fulfilled order offers an ERC20 item and requires an ERC721 item to the offerer and the same ERC20 item to another recipient with an amount less than or equal to the offered amount, the fulfiller does not need to own the ERC20 item as it will first be received from the offerer.
- 履行者应该在收到所有提供的物品后，对所有考虑项目有足够的余额——例如，如果履行的订单提供了 ERC20 物品，并且需要向提供者提供 ERC721 物品，而将相同的 ERC20 物品提供给另一个收件人，但金额更少大于或等于提供的金额，履行者不需要拥有 ERC20 项目，因为它将首先从提供者处收到。
- If the fulfiller does not elect to utilize a conduit, they need to have sufficient approvals set for the Seaport contract for all ERC20, ERC721, and ERC1155 consideration items on the fulfilled order.
- 如果履行者不选择使用管道，他们需要为已履行订单上的所有 ERC20、ERC721 和 ERC1155 对价项目的海港合同设置足够的批准。
- If the fulfiller _does_ elect to utilize a conduit, they need to have sufficient approvals set for their respective conduit for all ERC20, ERC721, and ERC1155 consideration items on the fulfilled order.
- 如果履行者_does_ 选择使用管道，他们需要为已履行订单上的所有 ERC20、ERC721 和 ERC1155 考虑项目为其各自的管道设置足够的批准。
- If the fulfilled order specifies Ether (or other native tokens) as consideration items, the fulfiller must be able to supply the sum total of those items as `msg.value`.
- 如果已履行的订单将以太币（或其他原生代币）指定为考虑项目，则履行者必须能够以“msg.value”的形式提供这些项目的总和。


When fulfilling a set of _match_ orders, the following requirements need to be checked to ensure that the order will be fulfillable:
在履行一组_match_订单时，需要检查以下要求以确保订单可以履行：


- Each account that sources the ERC20, ERC721, or ERC1155 item for an execution that will be performed as part of the fulfillment must have sufficient balance and approval on Seaport or the indicated conduit at the time the execution is triggered. Note that prior executions may supply the necessary balance for subsequent executions.
- 为将作为履行的一部分执行的执行采购 ERC20、ERC721 或 ERC1155 项目的每个帐户必须在触发执行时在 Seaport 或指定的渠道上具有足够的余额和批准。请注意，先前的执行可能会为后续执行提供必要的平衡。
- The sum total of all executions involving Ether (or other native tokens) must be supplied as `msg.value`. Note that executions where the offerer and the recipient are the same account will be filtered out of the final execution set.
- 涉及以太币（或其他原生代币）的所有执行的总和必须以“msg.value”的形式提供。请注意，提供者和接收者是同一帐户的执行将从最终执行集中被过滤掉。


### Partial Fills
### 部分填充


When constructing an order, the offerer may elect to enable partial fills by setting an appropriate order type. Then, orders that support partial fills can be fulfilled for some _fraction_ of the respective order, allowing subsequent fills to bypass signature verification. To summarize a few key points on partial fills:
在构建订单时，报价方可以选择通过设置适当的订单类型来启用部分成交。然后，支持部分执行的订单可以为相应订单的某些_fraction_执行，允许后续执行绕过签名验证。总结一下部分填充的几个关键点：


- When creating orders that support partial fills or determining a fraction to fill on those orders, all items (both offer and consideration) on the order must be cleanly divisible by the supplied fraction (i.e. no remainder after division).
- 当创建支持部分执行的订单或确定要执行这些订单的分数时，订单上的所有项目（报价和对价）必须能被提供的分数完全整除（即除法后没有余数）。
- If the desired fraction to fill would result in more than the full order amount being filled, that fraction will be reduced to the amount remaining to fill. This applies to both partial fill attempts as well as full fill attempts. If this behavior is not desired (i.e. the fill should be "all or none"), the fulfiller can either use a "basic" order method if available (which requires that the full order amount be filled), or use the "match" order method and explicitly provide an order that requires the full desired amount be received back.
- 如果要填写的所需部分会导致已填写的订单数量超过全部订单金额，则该部分将减少为剩余要填写的数量。这适用于部分填充尝试和完全填充尝试。如果不需要这种行为（即填充应该是“全部或无”），履行者可以使用“基本”订单方法（如果可用）（要求填写全部订单金额），或使用“匹配” order 方法，并明确提供一个要求收到全部所需金额的订单。
- By way of example: if one fulfiller tries to fill 1/2 of an order but another fulfiller first fills 3/4 of the order, the original fulfiller will end up filling 1/4 of the order.
- 举例来说：如果一个履行者尝试履行订单的 1/2，但另一个履行者首先履行订单的 3/4，则原始履行者最终将履行订单的 1/4。
- If any of the items on a partially fillable order specify a different "startAmount" and "endAmount (e.g. they are ascending-amount or descending-amount items), the fraction will be applied to _both_ amounts prior to determining the current price. This ensures that cleanly divisible amounts can be chosen when constructing the order without a dependency on the time when the order is ultimately fulfilled.
- 如果部分可成交订单上的任何项目指定了不同的“startAmount”和“endAmount”（例如，它们是递增数量或递减数量的项目），则在确定当前价格之前，分数将应用于_both_数量。这确保在构建订单时可以选择完全可分割的金额，而不依赖于最终完成订单的时间。
- Partial fills can be combined with criteria-based items to enable constructing orders that offer or receive multiple items that would otherwise not be partially fillable (e.g. ERC721 items).
- 部分填写可以与基于标准的项目结合使用，以支持构建提供或接收多个项目的订单，否则这些项目将无法部分填写（例如 ERC721 项目）。


- By way of example: an offerer can create a partially fillable order to supply up to 10 ETH for up to 10 ERC721 items from a given collection; then, any fulfiller can fill a portion of that order until it has been fully filled (or cancelled).
- 举个例子：提供者可以创建一个部分可成交的订单，为给定集合中最多 10 个 ERC721 项目提供最多 10 个 ETH；然后，任何履行者都可以履行该订单的一部分，直到它被完全履行（或取消）。


## Sequence of Events
## 事件顺序


### Fulfill Order
### 履行订单


When fulfilling an order via `fulfillOrder` or `fulfillAdvancedOrder`:
通过 `fulfillOrder` 或 `fulfillAdvancedOrder` 完成订单时：


1. Hash order
1.哈希顺序
- Derive hashes for offer items and consideration items
- 为报价项目和考虑项目派生哈希
- Retrieve current nonce for the offerer
- 检索提供者的当前随机数
- Derive hash for order
- 为订单派生哈希
2. Perform initial validation
2. 执行初始验证
- Ensure current time is inside order range
- 确保当前时间在订单范围内
- Ensure valid caller for the order type; if the order type is restricted and the caller is not the offerer or the zone, call the zone to determine whether the order is valid
- 确保订单类型的调用者有效；如果订单类型受限且调用者不是offerer或者zone，调用zone判断订单是否有效
3. Retrieve and update order status
3.检索和更新订单状态
- Ensure order is not cancelled
- 确保订单不会被取消
- Ensure order is not fully filled
- 确保订单未满
- If the order is _partially_ filled, reduce the supplied fill amount if necessary so that the order is not overfilled
- 如果订单_部分_成交，必要时减少提供的成交量，以免订单被超额成交
- Verify the order signature if not already validated
- 如果尚未验证，请验证订单签名
- Determine fraction to fill based on preference + available amount
- 根据偏好 + 可用数量确定要填充的分数
- Update order status (validated + fill fraction)
- 更新订单状态（已验证 + 填充率）
4. Determine amount for each item
4.确定每个项目的金额
- Compare start amount and end amount
- 比较起始金额和结束金额
- if they are equal: apply fill fraction to either one, ensure it divides cleanly, and use that amount
- 如果它们相等：将填充分数应用于任何一个，确保它干净地划分，并使用该数量
- if not: apply fill fraction to both, ensuring they both divide cleanly, then find linear fit based on current time
- 如果不是：对两者应用填充分数，确保它们都干净地划分，然后根据当前时间找到线性拟合
5. Apply criteria resolvers
5. 应用标准解析器
- Ensure each criteria resolver refers to a criteria-based order item
- 确保每个标准解析器都引用基于标准的订单项
- Ensure the supplied identifier for each item is valid via inclusion proof if the item has a non-zero criteria root
- 如果项目具有非零标准根，则通过包含证明确保为每个项目提供的标识符有效
- Update each item type and identifier
- 更新每个项目类型和标识符
- Ensure all remaining items are non-criteria-based
- 确保所有剩余项目均不基于标准
6. Emit OrderFulfilled event
6. 发出 OrderFulfilled 事件
- Include updated items (i.e. after amount adjustment and criteria resolution)
- 包括更新的项目（即在金额调整和标准解决之后）
7. Transfer offer items from offerer to caller
7. 将要约项目从要约人转移到来电者
- Use either conduit or Seaport directly to source approvals, depending on order type
- 根据订单类型，直接使用管道或海港获得批准
8. Transfer consideration items from caller to respective recipients
8. 将考虑项目从呼叫者转移到相应的接收者
- Use either conduit or Seaport directly to source approvals, depending on the fulfiller's stated preference
- 使用管道或海港直接获得批准，具体取决于履行者声明的偏好


> Note: `fulfillBasicOrder` works in a similar fashion, with a few exceptions: it reconstructs the order from a subset of order elements, skips linear fit amount adjustment and criteria resolution, requires that the full order amount be fillable, and performs a more minimal set of transfers by default when the offer item shares the same type and token as additional consideration items.
> 注意：`fulfillBasicOrder` 以类似的方式工作，但有一些例外：它从订单元素的子集重建订单，跳过线性拟合量调整和标准解析，要求完整的订单量是可填写的，并执行更多当报价项目与附加考虑项目共享相同类型和令牌时，默认情况下的最小转移集。


### Match Orders
### 匹配订单


When matching a group of orders via `matchOrders` or `matchAdvancedOrders`, steps 1 through 6 are nearly identical but are performed for _each_ supplied order. From there, the implementation diverges from standard fulfillments:
当通过 `matchOrders` 或 `matchAdvancedOrders` 匹配一组订单时，步骤 1 到 6 几乎相同，但针对_每个_提供的订单执行。从那里开始，实现与标准实现不同：


7. Apply fulfillments
7. 应用履行
- Ensure each fulfillment refers to one or more offer items and one or more consideration items, all with the same type and token, and with the same approval source for each offer item and the same recipient for each consideration item
- 确保每次履行涉及一个或多个报价项目和一个或多个考虑项目，所有这些项目都具有相同的类型和令牌，并且每个报价项目具有相同的批准源和每个考虑项目的相同收件人
- Reduce the amount on each offer item and each consideration item to zero and track total reduced amounts for each
- 将每个报价项目和每个考虑项目的金额减少到零，并跟踪每个项目的总减少金额
- Compare total amounts for each and add back the remaining amount to the first item on the appropriate side of the order
- 比较每个的总金额，并将剩余金额加回订单适当一侧的第一个项目
- Return a single execution for each fulfillment
- 为每个履行返回一个执行
8. Scan each consideration item and ensure that none still have a nonzero amount remaining
8. 扫描每个考虑项目并确保没有一个仍然有非零的剩余金额
9. Perform transfers as part of each execution
9. 作为每次执行的一部分执行传输
- Use either conduit or Seaport directly to source approvals, depending on the original order type
- 根据原始订单类型，直接使用管道或海港获得批准
- Ignore each execution where `to == from` or `amount == 0` _(NOTE: the current implementation does not perform this last optimization)_
- 忽略 `to == from` 或 `amount == 0` 的每次执行 _（注意：当前实现不执行最后一次优化）_


## Known Limitations and Workarounds
## 已知限制和解决方法


- As all offer and consideration items are allocated against one another in memory, there are scenarios in which the actual received item amount will differ from the amount specified by the order — notably, this includes items with a fee-on-transfer mechanic. Orders that contain items of this nature (or, more broadly, items that have some post-fulfillment state that should be met) should leverage "restricted" order types and route the order fulfillment through a zone contract that performs the necessary checks after order fulfillment is completed.
- 由于所有报价和对价项目都在内存中相互分配，因此在某些情况下，实际收到的项目数量将与订单指定的数量不同- 特别是，这包括具有转移费用机制的项目。包含此类商品的订单（或更广泛地说，应满足某些履行后状态的商品）应利用“受限”订单类型，并通过区域合同路由订单履行，该合同在订单履行后执行必要的检查完成了。
- As all offer items are taken directly from the offerer and all consideration items are given directly to the named recipient, there are scenarios where those accounts can increase the gas cost of order fulfillment or block orders from being fulfilled outright depending on the item being transferred. If the item in question is Ether or a similar native token, a recipient can throw in the payable fallback or even spend excess gas from the submitter. Similar mechanics can be leveraged by both offerers and receives if the item in question is a token with a transfer hook (like ERC1155 and ERC777) or a non-standard token implementation. Potential remediations to this category of issue include wrapping Ether as WETH as a fallback if the initial transfer fails and allowing submitters to specify the amount of gas that should be allocated as part of a given fulfillment. Orders that support explicit fulfillments can also elect to leave problematic or unwanted offer items unspent as long as all consideration items are received in full.
- 由于所有报价项目都直接从报价方获取，所有对价项目都直接提供给指定的收件人，因此在某些情况下，这些帐户可能会增加订单履行的 gas 成本或阻止订单完全履行，具体取决于被转移的项目.如果有问题的项目是以太币或类似的原生代币，接收者可以投入应付的后备，甚至从提交者那里花费多余的气体。如果所讨论的项目是带有传输挂钩的代币（如 ERC1155 和 ERC777）或非标准代币实现，则提供者和接收者都可以利用类似的机制。对此类问题的潜在补救措施包括在初始传输失败时将 Ether 包装为 WETH 作为后备，并允许提交者指定应分配的气体量作为给定履行的一部分。支持明确履行的订单也可以选择不使用有问题或不需要的报价项目，只要全部收到所有对价项目。
- As fulfillments may be executed in whatever sequence the fulfiller specifies as long as the fulfillments are all executable, as restricted orders are validated via zones prior to execution, and as orders may be combined with other orders or have additional consideration items supplied, any items with modifiable state are at risk of having that state modified during execution if a payable Ether recipient or onReceived 1155 transfer hook is able to modify that state. By way of example, imagine an offerer offers WETH and requires some ERC721 item as consideration, where the ERC721 should have some additional property like not having been used to mint some other ERC721 item. Then, even if the offerer enforces that the ERC721 have that property via a restricted order that checks for the property, a malicious fulfiller could include a second order (or even just an additional consideration item) that uses the ERC721 item being sold to mint before it is transferred to the offerer. One category of remediation for this problem is to use restricted orders that do not implement `isValidOrder` and actually require that order fulfillment is routed through them so that they can perform post-fulfillment validation. Another interesting solution to this problem that retains order composability is to "fight fire with fire" and have the offerer include a "validator" ERC1155 consideration item on orders that require additional assurances; this would be a contract that contains the ERC1155 interface but is not actually an 1155 token, and instead leverages the `onReceived` hook as a means to validate that the expected invariants were upheld, reverting the "transfer" if the check fails (so in the case of the example above, this hook would ensure that the offerer was the owner of the ERC721 item in question and that it had not yet been used to mint the other ERC721). The key limitation to this mechanic is the amount of data that can be supplied in-band via this route; only three arguments ("from", "identifier", and "amount") are available to utilize.
- 由于履行可以按照履行者指定的任何顺序执行，只要履行都是可执行的，因为受限订单在执行之前通过区域验证，并且由于订单可以与其他订单合并或提供额外的考虑项目，所以任何项目如果应付 Ether 接收者或 onReceived 1155 传输挂钩能够修改该状态，则具有可修改状态的状态在执行期间有可能修改该状态。举例来说，假设提供者提供 WETH 并需要一些 ERC721 项目作为考虑，其中 ERC721 应该具有一些额外的属性，例如没有用于铸造其他 ERC721 项目。然后，即使报价方通过检查财产的受限订单强制 ERC721 拥有该财产，恶意履行者也可能包括使用之前出售给铸币厂的 ERC721 项目的第二个订单（甚至只是一个额外的考虑项目）它被转移给要约人。针对此问题的一类补救措施是使用未实现“isValidOrder”的受限订单，并且实际上需要通过它们路由订单履行，以便他们可以执行履行后验证。保留订单可组合性的另一个有趣的解决方案是“以火攻克”，并让报价者在需要额外保证的订单上包含“验证器”ERC1155 考虑项；这将是一个包含 ERC1155 接口但实际上不是 1155 令牌的合约，而是利用 `onReceived` 钩子作为验证预期不变量是否得到支持的手段，如果检查失败则恢复“转移”（所以在在上面的例子中，这个钩子将确保提供者是相关 ERC721 项目的所有者，并且它还没有被用来铸造另一个 ERC721）。这种机制的主要限制是可以通过此路由在带内提供的数据量；只有三个参数（“from”、“identifier”和“amount”）可供使用。
- As all consideration items are supplied at the time of order creation, dynamic adjustment of recipients or amounts after creation (e.g. modifications to royalty payout info) is not supported. However, a zone can enforce that a given restricted order contains _new_ dynamically computed consideration items by deriving them and either supplying them manually or ensuring that they are present via `isValidZoneIncludingExtraData` since consideration items can be extended arbitrarily, with the important caveat that no more than the original offer item amounts can be spent.
- 由于所有对价项目均在创建订单时提供，因此不支持在创建后动态调整收件人或金额（例如修改版税支付信息）。然而，一个区域可以强制一个给定的受限订单包含_new_动态计算的考虑项目，方法是派生它们并手动提供它们或确保它们通过`isValidZoneIncludingExtraData`存在，因为考虑项目可以任意扩展，重要的警告是不再有比原来的优惠项目金额可以花费。
- As all criteria-based items are tied to a particular token, there is no native way to construct orders where items specify cross-token criteria. Additionally, each potential identifier for a particular criteria-based item must have the same amount as any other identifier.
- 由于所有基于标准的项目都与特定令牌相关联，因此没有本地方式来构建项目指定跨令牌标准的订单。此外，基于特定标准的项目的每个潜在标识符必须与任何其他标识符具有相同的数量。
- As orders that contain items with ascending or descending amounts may not be filled as quickly as a fulfiller would like (e.g. transactions taking longer than expected to be included), there is a risk that fulfillment on those orders will supply a larger item amount, or receive back a smaller item amount, than they intended or expected. One way to prevent these outcomes is to utilize `matchOrders`, supplying a contrasting order for the fulfiller that explicitly specifies the maximum allowable offer items to be spent and consideration items to be received back. Special care should be taken when handling orders that contain both brief durations as well as items with ascending or descending amounts, as realized amounts may shift appreciably in a short window of time.
- 由于包含金额递增或递减的项目的订单可能不会像履行者希望的那样快速履行（例如，交易花费的时间比预期的要长），这些订单的履行可能会提供更大的项目数量，或收到比他们预期或预期的更少的物品数量。防止这些结果的一种方法是使用“matchOrders”，为履行者提供一个对比订单，明确指定要花费的最大允许报价项目和要收回的考虑项目。在处理包含短期持续时间以及金额递增或递减的项目时，应特别小心，因为已实现的金额可能会在短时间内发生明显变化。
- As all items on orders supporting partial fills must be "cleanly divisible" when performing a partial fill, orders with multiple items should to be constructed with care. A straightforward heuristic is to start with a "unit" bundle (e.g. 1 NFT item A, 3 NFT item B, and 5 NFT item C for 2 ETH) then applying a multiple to that unit bundle (e.g. 7 of those units results in a partial order for 7 NFT item A, 21 NFT item B, and 35 NFT item C for 14 ETH).
- 由于支持部分成交的订单上的所有项目在执行部分成交时必须是“完全可分割的”，因此应谨慎构建具有多个项目的订单。一个简单的启发式方法是从一个“单元”捆绑包开始（例如 1 个 NFT 项目 A、3 个 NFT 项目 B 和 5 个 NFT 项目 C 用于 2 ETH），然后将倍数应用于该单元捆绑包（例如，这些单元中的 7 个导致7 NFT 项目 A、21 NFT 项目 B 和 35 NFT 项目 C 的部分订单（14 ETH）。
- As Ether cannot be "taken" from an account, any order that contains Ether or other native tokens as an offer item (including "implied" mirror orders) must be supplied by the caller executing the order(s) as msg.value. This also explains why there are no `ERC721_TO_ETH` and `ERC1155_TO_ETH` basic order route types, as Ether cannot be taken from the offerer in these cases. One important takeaway from this mechanic is that, technically, anyone can supply Ether on behalf of a given offerer (whereas the offerer themselves must supply all other items). It also means that all Ether must be supplied at the time the order or group of orders is originally called (and the amount available to spend by offer items cannot be increased by an external source during execution as is the case for token balances).
- 由于不能从账户中“获取”以太币，任何包含以太币或其他原生代币作为报价项目的订单（包括“隐含”镜像订单）必须由执行订单的调用者作为 msg.value 提供。这也解释了为什么没有 `ERC721_TO_ETH` 和 `ERC1155_TO_ETH` 基本订单路由类型，因为在这些情况下无法从提供者那里获取以太币。这个机制的一个重要收获是，从技术上讲，任何人都可以代表给定的提议者提供以太币（而提议者自己必须提供所有其他物品）。这也意味着必须在最初调用订单或订单组时提供所有以太币（并且在执行期间，外部来源不能增加报价项目可花费的金额，就像代币余额的情况一样）。
- As extensions to the consideration array on fulfillment (i.e. "tipping") can be arbitrarily set by the caller, fulfillments where all matched orders have already been signed for or validated can be frontrun on submission, with the frontrunner modifying any tips. Therefore, it is important that orders fulfilled in this manner either leverage "restricted" order types with a zone that enforces appropriate allocation of consideration extensions, or that each offer item is fully spent and each consideration item is appropriately declared on order creation.
- 由于调用者可以任意设置履行时考虑数组的扩展（即“小费”），所有匹配订单已经签署或验证的履行可以在提交时抢先运行，领先者修改任何提示。因此，重要的是，以这种方式完成的订单要么利用具有强制适当分配对价扩展的区域的“受限”订单类型，要么每个报价项目都已完全用完，并且每个对价项目都在订单创建时适当声明。
- As orders that have been verified (via a call to `validate`) or partially filled will skip signature validation on subsequent fulfillments, orders that utilize EIP-1271 for verifying orders may end up in an inconsistent state where the original signature is no longer valid but the order is still fulfillable. In these cases, the offerer must explicitly cancel the previously verified order in question if they no longer wish for the order to be fulfillable.
- 由于已验证（通过调用 `validate`）或部分履行的订单将在后续履行时跳过签名验证，使用 EIP-1271 验证订单的订单可能最终处于原始签名不再存在的不一致状态有效，但订单仍可履行。在这些情况下，如果供应商不再希望订单可以履行，他们必须明确取消先前已验证的相关订单。
- As orders filled by the "fulfill available" method will only be skipped if those orders have been cancelled, fully filled, or are inactive, fulfillments may still be attempted on unfulfillable orders (examples include revoked approvals or insufficient balances). This scenario (as well as issues with order formatting) will result in the full batch failing. One remediation to this failure condition is to perform additional checks from an executing zone or wrapper contract when constructing the call and filtering orders based on those checks.
- 由于通过“可履行”方法履行的订单仅在这些订单已被取消、已完全履行或处于非活动状态时才会被跳过，因此仍可能对无法履行的订单尝试履行（例如，撤销批准或余额不足）。这种情况（以及订单格式问题）将导致整批失败。对此故障情况的一种补救措施是在构建调用时从执行区域或包装合约执行额外的检查，并根据这些检查过滤订单。
- As order parameters must be supplied upon cancellation, orders that were meant to remain private (e.g. were not published publically) will be made visible upon cancellation. While these orders would not be _fulfillable_ without a corresponding signature, cancellation of private orders without broadcasting intent currently requires the offerer (or the zone, if the order type is restricted and the zone supports it) to increment the nonce.
- 由于必须在取消时提供订单参数，因此本应保密的订单（例如未公开发布）将在取消时显示。虽然这些订单在没有相应签名的情况下不会_fulfillable_，但在没有广播意图的情况下取消私人订单目前需要提供者（或区域，如果订单类型受到限制并且区域支持它）增加随机数。
- As order fulfillment attempts may become public before being included in a block, there is a risk of those orders being front-run. This risk is magnified in cases where offered items contain ascending amounts or consideration items contain descending amounts, as there is added incentive to leave the order unfulfilled until another interested fulfiller attempts to fulfill the order in question. Remediation efforts include utilization of a private mempool (e.g. flashbots) and/or restricted orders where the respective zone enforces a commit-reveal scheme.
- 由于订单履行尝试可能会在被包含在区块中之前公开，因此这些订单存在被抢先交易的风险。如果提供的商品包含递增金额或对价商品包含递减金额，则这种风险会被放大，因为在另一个感兴趣的履行者尝试履行相关订单之前，会有额外的动机让订单未履行。补救措施包括使用私人内存池（例如 flashbots）和/或限制订单，其中各个区域执行提交-显示方案。
