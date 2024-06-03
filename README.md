
# Chat Application

This is a simple chat application consisting of a chat server and a chat client. The server handles multiple clients and facilitates communication between them.

## Data Structures

### Client
```c
typedef struct {
    int id;           // 5-digit unique id
    int socketfd;     // socket fd of client
    int grpCount;     // number of groups client has joined
    int joinedGroups[]; // group ids of groups that client has joined
} Client;
```

### Group
```c
typedef struct {
    int groupid;          // 3-digit unique id
    int membersid[5];     // members of group
    int tempMembers[4];   // pending member requests
    bool isCreated;       // group is active or not
    int grpsize;          // no. of members
    int reqSize;          // no. of request sent
    int admins[5];        // id of admins
    int adminlen;         // no. of admins
    bool isBroadcast;     // admin-only or not
} Group;
```

### AdminRequest
```c
typedef struct {
    int grpid;        // group id
    int posCount;     // approvals
    int negCount;     // denials
    int totalAdmins;  // total number of admins
    int memberid;     // requester id
    bool status;      // status of admin approval
} AdminRequest;
```

## Execution Flow

### Client
1. Client will request a connection to the server.
2. If the connection limit is exceeded, the connection will not be established.
3. Otherwise, the connection will be created.
4. A child process will be forked to handle user input and send requests to the server.
5. The parent process will receive the response from the server and print it on the console.

### Server
1. The server will occupy a socket and create a `fd_set` to store all `readfds`.
2. In each iteration, the server will check which sockets are ready for read operations.
3. If the current socket is of the server, it is ready to handle new client connection requests.
    - Create a new client record and push it to global records using a unique 5-digit ID. Add the client to the record using module hashing.
    - If the client limit is exceeded, send a termination response to the client and clear the socket assigned to that client.
4. Otherwise, read the request from the client and call `performAction` function to handle requests.

## Modules

### Commands and Their Functions

#### `/active`
- Iterate over clients array and print all the clients in the array.

#### `/send`
- Extract client id and message from request.
- Find socket of client from records.
- Send message to that socket fd.

#### `/broadcast`
- Iterate over all clients in records, get their socket fd, and send message to them.

#### `/makegroup`
- Extract all client ids from request.
- Generate a unique id for the group.
- Add members in members array of the group.
- Initialize default variables of the group.
- Make group as an active group.
- Send message to all group members.

#### `/makegroupreq`
- Extract all client ids from request.
- Generate a unique id for the group.
- Send request to clients to join the group.
- Store request count and client ids to whom the request has been sent.
- Mark group as inactive for now.

#### `/joingroup`
- Check whether request is sent to this client or not.
- If yes, then add them to the group by checking group limit.
- Send a confirmation message.
- Decrease the request count and remove client id from pending request array.

#### `/declinergroup`
- Check whether request is sent to this client or not.
- Decrease the request count and remove client id from pending request array.
- Send a confirmation message to admin about decline request.

#### `/makegroupbroadcast`
- Enable the toggle variable of group structure for broadcast.

#### `/sendgroup`
- Extract message from request.
- Iterate over client ids and send message to all using their socket fd.

#### `/addadmin`
- Check whether group id and client id exist or not.
- Check whether they are already admin or not.
- Check whether caller client is admin or not.
- Add the client to admin array and increment admin count.

#### `/addtogroup`
- Check whether group id and client id exist or not.
- Check whether caller client is admin or not.
- Add the client to members array and increment group size.

#### `/removefromgroup`
- Check whether group id and client id exist or not.
- Check whether caller client is admin or not.
- Remove the client from members array and decrement group size.
- Check whether group is empty, if yes then deactivate the group.

#### `/activegroups`
- Iterate over all groups and check whether this client is a member or not.
- If yes, print details of group.

#### `/makeadminreq`
- Create a separate structure to store admin requests.
- Add group id, requester id, admin count of that group.
- Send approval request to each admin of group.

#### `/approveadminreq`
- Extract group id and client id from request and check whether the request for admin exists or not.
- If yes, increase the positive count by 1.
- Check whether response from all admins has been received, if yes then change the status of request according to majority and send message to client about status.

#### `/declineadminreq`
- Extract group id and client id from request and check whether the request for admin exists or not.
- If yes, increase the negative count by 1.
- Check whether response from all admins has been received, if yes then change the status of request according to majority and send message to client about status.

#### `/quit`
- Remove client from clients array.
- Remove client from all groups they are part of.
- Check whether group is empty, if yes then deactivate the group.
- Remove it from master `fd_set`.

### Ctrl+C & Ctrl+Z Handler in Client
- Redirected the handler function to a user-defined function which sends `/quit` command to the server instead of killing the process.

## How to Run

### Server

To compile and run the server, use the following commands:

```sh
gcc ChatServer.c -o server
./server
```

### Client

To compile and run the client, use the following commands:

```sh
gcc ChatClient.c -o client
./client
```

## Notes

- Ensure that the server is running before starting any clients.
- The server and client should be run in the same network for proper communication.

## Troubleshooting

- If you encounter any issues, check that your GCC installation is correct and that there are no syntax errors in the source files.
- Make sure the server is running and listening on the correct port before starting the client.

## Contributing

If you would like to contribute to this project, please fork the repository and create a pull request with your changes.

## License

This project is licensed under the MIT License. See the LICENSE file for more details.

## Acknowledgments

- Thanks to the open-source community for providing tools and resources.
