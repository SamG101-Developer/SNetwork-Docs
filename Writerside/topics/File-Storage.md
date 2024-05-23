# File Storage

**_NB: This is a work in progress, may not get implemented._**

Whilst an FTP server can be setup as a Hidden Service, it is not the only way to store files on the network. Hidden
Services ensure control over resource, and end-to-end anonymity, but do not guarantee availability. Files can be stored
in a distributed manner, similar to how nodes are stored on the network. This allows for files to be stored on multiple
nodes, ensuring availability.

As these files can be seen by whoever is hosting them, the Provider Nodes do not need to remain anonymous for this
feature. This simplifies the process of storing files on the network, as the Provider Node does not need to set up an
onion route or use Broker Nodes.

Accessor Nodes still use onion routes to access the files, but the files are stored on multiple nodes, so the Accessor
Node can access the file from any of the nodes that store it. This ensures that the file is always available, even if
some of the nodes storing the file are offline.

Files are immutable, so once a file is stored on the network, it cannot be changed. If a file needs to be updated, a new
file must be created with the updated content. This new file will have a new ID, and the old file will still be
accessible on the network. File versions can be linked, so that the latest version can be easily found.

This also helps authenticate the 
