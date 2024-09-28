## Context
To change the default namespace more permanently, context is used. This gets recorded in a kubectl configuration file, usually located at $HOME/.kube/config. This configuration file also stores how to
both find and authenticate the cluster. 
To create a context with a different default namespace for kubectl commands use:
*$ kubectl config set-context my-context --namespace=kube-tut*
This creates a new context, but it doesn’t actually start using it yet. To use this newly created context,run:
*$ kubectl config use-context my-context*
Contexts can also be used to manage different clusters or different users for authenticating to those clusters using the --users or --clusters flags with the set-context command.