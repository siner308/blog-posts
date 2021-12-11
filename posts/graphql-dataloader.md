---
layout: post
title:  "NestJS Dataloader 적용해보기"
subtitle: "GraphQL에서 발생하는 N+1 문제 해결방법"
author: "Siner"
catalog: true
header-mask:  0.3
tags:
    - graphql
    - nestjs
    - typescript
date:   2021-12-11
multilingual: flase
---

- 이 포스팅의 예제 코드는 typescript + nestjs를 기반으로 작성되었지만, 다른 언어에도 마찬가지로 적용이 가능합니다.
- 이 포스팅은 아래의 스키마를 기반으로 작성되었습니다.

```graphql
type Order {
  id: Int!
  user: User!
  userId: Int!
  items: [OrderItem]!
}

type User {
  id: Int!
}

type OrderItem {
  id: Int!
  orderId: Int!
}
```

```graphql
type Query orders {
  orders {
    id
    user {
      id
    }
    items {
      id
    }
  }
}
```
---

# Dataloader 적용 전

위의 쿼리 요청을 위해 서버에서 Resolver를 아래와 같이 작성했다고 가정해봅시다.
아직 Dataloader를 적용하지 않은 상태입니다.

```typescript
// order.resolver.ts
@Resolver(() => Order)
class OrderResolver {
  constructor(
    private readonly orderRepo: OrderRepo,
    private readonly userRepo: UserRepo,
    private readonly orderItemRepo: OrderItemRepo,
  )

  @Query(() => [Order])
  orders(): Promise<Order[]> {
    return this.orderRepo.findAll();
  }

  @ResolveField(() => User)
  user(@Parent(): order: Order): Promise<User> {
    return this.userRepo.getById(order.userId);
  }

  @ResolveField(() => [OrderItem])
  orderItems(@Parent(): order: Order): Promise<OrderItem[]> {
    return this.orderItemRepo.findByOrderId(order.id);
  }
}
```

만약 orders 쿼리의 결과로 5개의 주문이 반환된다면, Dataloader 작업을 하지 않은 경우 아래처럼 쿼리가 발생합니다.
(5개의 주문 id는 1,2,3,4,5이며 주문한 유저의 id 또한 각각 1,2,3,4,5라고 가정해봅시다.)

```sql
SELECT * FROM order;

SELECT * FROM user WHERE id=1;
SELECT * FROM user WHERE id=2;
SELECT * FROM user WHERE id=3;
SELECT * FROM user WHERE id=4;
SELECT * FROM user WHERE id=5;

SELECT * FROM order_item WHERE order_id=1;
SELECT * FROM order_item WHERE order_id=2;
SELECT * FROM order_item WHERE order_id=3;
SELECT * FROM order_item WHERE order_id=4;
SELECT * FROM order_item WHERE order_id=5;
```

user와 order_item 데이터를 가져오기 위해 반환된 order 수 만큼의 쿼리가 추가적으로 발생했습니다. 이를 N+1 Problem 이라고 부릅니다.

REST API의 경우 모든 데이터를 한번에 가공하여 클라이언트에 전달하지만, GraphQL의 각각의 type은 해당 값을 resolve 하기위한 단일 목적만을 중점으로 두게 됩니다. 따라서 해당 type의 자식으로 있는 데이터의 경우 부모(Parent)로부터 정보를 받아서 쿼리 요청을 하게 되는데, 만약 부모가 리스트 형태로 조회되는 쿼리였을 경우 (ex. orders) 각각의 부모에서 자식 데이터를 가져오기 위한 쿼리가 발생하여 N+1 쿼리 문제가 발생하게 되는데, 이를 해결하기 위해서 Dataloader를 사용합니다. 

Dataloader는 batching과 caching을 통해 백엔드 부하를 줄여줍니다.

# Dataloader 적용

부모와 1:1 관계를 가지는 User 데이터의 경우 아래처럼 Dataloader를 작성할 수 있습니다.

```typescript
// user.loader.ts
export class UserLoader {
  getByUserId: Dataloader<number, User>  = new Dataloader<number, User>(
    async (userIds: number[]) => {
      const users: User[] = await this.userRepo.findByIds(userIds);
      return userIds.map((userId) => users.find((user) => user.id === userId));
    };
  ),
}
```

부모와 1:N 관계를 가지는 OrderItem 데이터의 경우 아래처럼 Dataloader를 작성할 수 있습니다.

```typescript
// order-item.loader.ts
export class OrderItemLoader {
  findByOrderId: Dataloader<number, OrderItem[]> = new Dataloader<number, OrderItem[]>(
    async (orderIds: number[]) => {
      const orderItems: OrderItem[] = await this.orderItemRepo.findByOrderIds(orderIds);
      const orderItemGroup: { [key: number]: OrderItem[] } = {};
      orderItems.forEach((orderItem: OrderItem) => {
        if (!orderItemGroup[orderItem.orderId]) {
          orderItemGroup[orderItem.orderId] = [];
        }
        orderItemGroup[orderItem.orderId].push(orderItem);
      });
      return orderIds.map((orderId: number) => orderItemGroup[orderId]);
    },
  );
}
```

위 Dataloader를 아까 작성했던 Resolver에 적용하면 아래와 같습니다.

```typescript
// order.resolver.ts
@Resolver(() => Order)
class OrderResolver {
  constructor(
    private readonly orderRepo: OrderRepo,
    private readonly userLoader: UserLoader,
    private readonly orderItemLoader: OrderItemLoader,
  )

  @Query(() => [Order])
  orders(): Promise<Order[]> {
    return this.orderRepo.findAll();
  }

  @ResolveField(() => User)
  user(@Parent(): order: Order): Promise<User> {
    return this.userLoader.getByUserId(order.userId);
  }

  @ResolveField(() => [OrderItem])
  orderItems(@Parent(): order: Order): Promise<OrderItem[]> {
    return this.orderItemLoader.findByOrderId(order.id);
  }
}
```

Dataloader 적용 후 쿼리를 보면 아래와 같습니다.

```sql
SELECT * FROM order;

SELECT * FROM user WHERE id IN (1,2,3,4,5);

SELECT * FROM order_item WHERE order_id IN (1,2,3,4,5);
```

이로써 N+1 문제가 해결되어 백엔드 부하가 줄어들게 됩니다.

dataloader 구현체는 코드가 그렇게 길지 않으니 직접 확인해보는것도 나쁘지 않은 것 같습니다.
[https://github.com/graphql/dataloader](https://github.com/graphql/dataloader)
