---
eip: 9999
title: Symmetric Blob Base Fee Adjustment Mechanism
description: Adjust the blob base fee mechanism to ensure symmetric updates around the target blob gas per block
author: Toni Wahrstätter (@nerolation)
discussions-to: TBD
status: Draft
type: Standards Track
category: Core
created: 2024-10-31
---

## Abstract

This EIP proposes an adjustment to the blob base fee mechanism proposed in [EIP-4844](./eip-4844.md) to ensure symmetric scaling of the base fee, regardless of the target blob gas per block relative to the maximum blob gas per block. The current mechanism maintains symmetry only when the target is exactly half of the maximum. By introducing a scaling factor in the calculation of `excess_blob_gas`, we achieve consistent and predictable fee adjustments, enhancing the network's efficiency and economic fairness.

## Motivation

The motivation for this proposal is to:

- **Ensure Symmetry**: Maintain symmetric maximum base fee changes, regardless of the target and maximum blob gas settings.
- **Enhance Predictability**: Provide consistent fee adjustments to improve network stability and predictability for users and validators.
- **Accommodate Flexibility**: Allow for flexible target and maximum blob gas configurations without compromising the fee adjustment mechanism.

## Specification

The current formula for determining the excess blob gas is defined as follows_
```python
def calc_excess_blob_gas(parent: Header) -> int:
    if parent.excess_blob_gas + parent.blob_gas_used < TARGET_BLOB_GAS_PER_BLOCK:
        return 0
    else:
        return parent.excess_blob_gas + parent.blob_gas_used - TARGET_BLOB_GAS_PER_BLOCK
```

The `excess_blob_gas` calculation is modified to include a scaling factor that adjusts for asymmetry when the target is not half of the maximum. The new calculation is as follows:

```python
def calc_excess_blob_gas(parent: Header) -> int:
    total_blob_gas = parent.excess_blob_gas + parent.blob_gas_used
    blob_gas_delta = total_blob_gas - TARGET_BLOB_GAS_PER_BLOCK
    if blob_gas_delta <= 0:
        return 0
    blob_gas_scaling_factor = TARGET_BLOB_GAS_PER_BLOCK / (MAX_BLOB_GAS_PER_BLOCK - TARGET_BLOB_GAS_PER_BLOCK)
    excess_blob_gas = blob_gas_delta * blob_gas_scaling_factor
    return excess_blob_gas
```

## Rationale

In the current protocol, the blob base fee adjustment mechanism assumes that the target blob gas per block is exactly half of the maximum blob gas per block. This assumption ensures symmetric maximum base fee increases and decreases, maintaining a balance in network economics.

However, when the target deviates from being half of the maximum (e.g., a target of 4 blobs with a maximum of 6 blobs), asymmetry arises because the maximum possible deviation above the target is not equal to the maximum possible deviation below the target. Specifically:

- **Maximum Positive Deviation**: `MAX_BLOB_GAS_PER_BLOCK - TARGET_BLOB_GAS_PER_BLOCK`
- **Maximum Negative Deviation**: `TARGET_BLOB_GAS_PER_BLOCK`

This imbalance leads to unequal maximum percentage changes in the base fee:

- The maximum base fee increase (when the blob gas used equals the maximum) is smaller than the maximum base fee decrease (when the blob gas used is zero).
- This can result in inefficient fee market dynamics and unintended economic incentives within the network.

To correct this asymmetry, we introduce a scaling factor in the calculation of `excess_blob_gas`. The scaling factor adjusts positive deviations from the target so that, after scaling, the maximum positive and negative deviations are equal in magnitude.

The scaling factor is calculated as:

```python
blob_gas_scaling_factor = TARGET_BLOB_GAS_PER_BLOCK / (MAX_BLOB_GAS_PER_BLOCK - TARGET_BLOB_GAS_PER_BLOCK)
```

By applying this scaling factor to positive deviations (when `blob_gas_delta > 0`), we ensure:

Symmetric Maximum Deviations: The scaled maximum positive deviation equals the maximum negative deviation.
Consistent Base Fee Adjustments: The base fee can increase or decrease by the same percentage amount, maintaining economic fairness.

## Backwards Compatibility

This is a backwards incompatible fee mechanism change that requires a scheduled network upgrade.

## Security Considerations

No security concerns have been identified.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
