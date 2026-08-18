# GMX V2 Order Flow Research

## Overview

This document describes the GMX V2 order execution flows implemented and tested in this repository.

The project uses Foundry with an Arbitrum mainnet fork to study how GMX V2 perpetual trading orders move through the protocol from order creation to position execution.

The purpose is to understand the interaction between the ExchangeRouter, OrderVault, OrderHandler, oracle validation, and position management components.

---

## High-Level Order Flow

The simplified lifecycle tested in this project is:

```text
User
 |
 | createOrder()
 v
ExchangeRouter
 |
 | sendWnt()
 v
OrderVault
 |
 | order creation
 v
OrderHandler
 |
 | keeper execution
 v
Oracle Price Validation
 |
 v
Position / Position Update
