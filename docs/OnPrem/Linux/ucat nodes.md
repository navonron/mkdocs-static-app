# ucat nodes

`ucat` is a tool that updates changes that are happening in almost real time (for example: changing specialty / owner / operating system version / etc ).

Other tools takes longer to be updated.

Downside of `ucat` is that you have to execute it on the node you want to check.

```
ucat nodes key dnsdomain cores modelname owner location memory mode speciality | grep NODE_NAME
```