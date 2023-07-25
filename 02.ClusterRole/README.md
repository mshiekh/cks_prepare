# Clusterrole


*Create a new serviceaccount (sa02)

*Create a role named role02, and bind role02 through rolebinding, and only allow update operations on persistentvolumeclaims.

```shell
1. kubectl create sa sa01

2. kubectl create role role02 --verb=update --resource=persistentvolumeclaims

3. kubectl create rolebinding xxx-rolebinding --role role02 --serviceaccount namespace:sa02
```
