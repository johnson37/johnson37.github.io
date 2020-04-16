# tcp keep alive

tcp keep alive 机制是为了考虑到tcp server hang住的情况，在tcp一段时间不发送报文的情况下，client发送keepalive报文。如果server果真hang住，则关闭tcp连接。

