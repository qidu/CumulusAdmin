CumulusAdmin
============

CumulusAdmin is a database-backed administration tool for <a href="https://github.com/OpenRTMFP/Cumulus">Cumulus</a>, an open source <a href="http://www.adobe.com/products/flash-media-enterprise/rtmfp-faq.html">RTMFP</a> server.

The server side code has been written in LUA that is provided natively by Cumulus. The administration tool itself has been written in PHP and by default it is configured to be backed by a MySQL database. Though, other databases should work, too. We use it at <a href="http://www.openrtmfp.net">OpenRTMFP.net</a> for maintainance and hereby commit it to the Cumulus community for that it will be helpful.


Features
--------

* Manage developers and applications via a database
* Manage developers' keys
* Manage applications identified by a unique path
* Enable/disable certain developers and applications
* Configure if publishing is allowed for client->server streams
* If allowed, optionally set a publishing password
* Configure subscribe/unsubscribe callbacks for publisher->server->subscriber streams to notify the publisher about connected/disconnected subscribers
* Have a complete image of peers and publications inside the database, e.g. for live statistics


Examples for subscribe/unsubscribe callbacks
--------------------------------------------

* onRelayConnect(publicationName:String, connecedPeerId:String, totalSubscribers:Number):void
* onRelayDisconnect(publicationName:String, disconnectedPeerId:String, remainingSubscribers:Number):void
* These callbacks for example allow to dynamically start/stop a stream depending on the subscriber count, mostly to save server bandwidth

SQL scheme
----------

Table "developers":
* id:Number
* enabled:0|1
* key:String|NULL
* contact:String|NULL

Table "applications":
* id:Number
* enabled:0|1
* path:String
* developer_id:Number
* allow_publish:0|1
* publish_password:String|NULL
* subscribe_callback:String|NULL
* unsubscribe_callback:String|NULL

Table "peers":
* id:String
* application_id:Number
* address:String[host:port]
* pageUrl:String
* swfUrl:String
* flashVersion:String

Table "publications":
* application_id:Number
* peer_id:Number
* name:String
* subscribers:Number


Installation
------------
* Create the database. You can use the included sql/mysql.sql to set up the scheme.
* Copy the included lua/main.lua to CumulusServer/www/main.lua
* Edit the CumulusServer/www/main.lua to match your database configuration (database, user, pass, host, port)
* Create developers and applications inside the database. An example is included in sql/mysql-example.sql

Usage
-----
* To add a new developer, insert one into the "developers" table.
* To add a new application, insert one into the "applications" table and create the corresponding directory in CumulusServer/www including an empty main.lua. If you are going to add the application "/example", you just create the directory and main.lua as CumulusServer/www/example/main.lua
* To allow everyone to publish client->server streams, set the "allow_publish" field for the desired application to "1" and leave the "publish_password" empty.
* To allow everyone who knows the correct password to publish client->server streams, set the "allow_publish" field for the desired application to "1" and set a "publish_password"
* To not allow anyone to publish client->server streams, set the "alllow_publish" field to "0"
* To connect to an application, use:

```
// For an application with path="/example" associated to an developer with key="MyDeveloperKey"
var con:NetConnection = new NetConnection();
con.connect("/example", "MyDeveloperKey");
```

* To handle subscribe/unsubscribe callbacks, set the callback names for the application you want to use them with in the database and attach a custom client listener to your NetStream instance that you have connected with the NetStream.CONNECT_TO_FMS flag:

```
// ActionScript 3
var ns:NetStream = new NetStream(con, NetStream.CONNECT_TO_FMS);
var c:Object = new Object;

// For subscribe_callback="onRelayConnected" and unsubscribe_callback="onRelayDisconnected"
c.onRelayConnected = function(publicationName:String, peerId:String, total:Number):void {
	trace("Peer "+peerId+" connected to publication "+publicationName+" (now "+total+" total subscribers)");
}
c.onRelayDisconnected = function(publicationName:String, peerId:String, remaining:Number):void {
	trace("Peer "+peerId+" disconnected from publication "+publicationName+" (now "+remaining+" remaining subscribers)")
}
ns.client = c;
// ...

// For allow_publish="1"
ns.publish("somePublicationName");

// ...or if you have also also set publish_password="somePassword"
ns.publish("somePublicationName", "somePassword");
```

* To get an overview of your current peers and publication statistics simply query the "peers" and "publications" tables. These contain a complete image of the current state of CumulusServer.
* To enable/disable developers or applications, change the "enabled" property of the corresponding row in the database.


Things still to do
------------------

* Easy to use PHP administration frontend
* Extend subscribe/unsubscribe callbacks to let the publisher decide if a subscriber is allowed to connect
* Configuration via CumulusServer.ini
* Automatic refetching of developers and applications without the requirement for a server restart
* Decide what to do about the requirement to create empty directories with empty main.lua's for each application


License
-------
This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.
