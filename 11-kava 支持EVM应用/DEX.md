Metamask 中添加 http://10.100.42.143:8545/






dkava tx bank send whale kava1q0dkky0505r555etn6u2nz4h4kjcg5y8dg863a 1000000ukava --gas-prices 0.001ukava -y





dkava query bank balances kava15tmj37vh7ch504px9fcfglmvx6y9m70646ev8t




浏览区 http://10.100.42.143/ 



```bash
# Add an alias to the dockerized kava cli
alias dkava='docker exec -it generated-kavanode-1 kava'

# Confirm that the alias has been added
alias kava

# For versions before v0.16.x
alias dkvcli='docker exec -it generated-kavanode_1 kvcli'
```
‰