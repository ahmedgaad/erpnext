### Concept of FIFO Slots

Since we need to know age-wise remaining stock, we maintain all the inward entries as slots. So each time stock comes in, a slot is added for the same.

Eg. For Item A:
----------------------
Date | Qty | Queue
----------------------
1st  | +50 | [[50, 1-12-2021]]
2nd  | +20 | [[50, 1-12-2021], [20, 2-12-2021]]
----------------------

Now the queue can tell us the total stock and also how old the stock is.
Here, the balance qty is 70.
50 qty is (today-the 1st) days old
20 qty is (today-the 2nd) days old

> Note: We generate FIFO slots warehouse wise as stock reconciliations from different warehouses can cause incorrect values.
### Calculation of FIFO Slots

#### Case 1: Outward from sufficient balance qty
----------------------
Date | Qty | Queue
----------------------
1st  | +50 | [[50, 1-12-2021]]
2nd  | -20 | [[30, 1-12-2021]]
2nd  | +20 | [[30, 1-12-2021], [20, 2-12-2021]]

Here after the first entry, while issuing 20 qty:
- **since 20 is lesser than the balance**, **qty_to_pop (20)** is simply consumed from first slot (FIFO consumption)
- Any inward entry after as usual will get its own slot added to the queue

#### Case 2: Outward from sufficient cumulative (slots) balance qty
----------------------
Date | Qty | Queue
----------------------
1st  | +50 | [[50, 1-12-2021]]
2nd  | +20 | [[50, 1-12-2021], [20, 2-12-2021]]
2nd  | -60 | [[10, 2-12-2021]]

- Consumption happens slot wise. First slot 1 is consumed
- Since **qty_to_pop (60) is greater than slot 1 qty (50)**, the entire slot is consumed and popped
- Now the queue is [[20, 2-12-2021]], and **qty_to_pop=10** (remaining qty to pop)
- It then goes ahead to the next slot and consumes 10 from it
- Now the queue is [[10, 2-12-2021]]

#### Case 3: Outward from insufficient balance qty
> This case is possible only if **Allow Negative Stock** was enabled at some point/is enabled.

----------------------
Date | Qty | Queue
----------------------
1st  | +50 | [[50, 1-12-2021]]
2nd  | -60 | [[-10, 1-12-2021]]

- Since **qty_to_pop (60)** is more than the balance in slot 1, the entire slot is consumed and popped
- Now the queue is **empty**, and **qty_to_pop=10** (remaining qty to pop)
- Since we still have more to consume, we append the balance since 60 is issued from 50 i.e. -10.
- We register this negative value, since the stock issue has caused the balance to become negative

Now when stock is inwarded:
- Instead of adding a slot we check if there are any negative balances.
- If yes, we keep adding positive stock to it until we make the balance positive.
- Once the balance is positive, the next inward entry will add a new slot in the queue

Eg:
----------------------
Date | Qty | Queue
----------------------
1st  | +50 | [[50, 1-12-2021]]
2nd  | -60 | [[-10, 1-12-2021]]
3rd  | +5  | [[-5, 3-12-2021]]
4th  | +10 | [[5, 4-12-2021]]
4th  | +20 | [[5, 4-12-2021], [20, 4-12-2021]]