# VotifierAndListener

VotifierAndListener (Val) is a Granite plugin whose purpose is to be notified (aka *votified*) when a vote is made on a Minecraft server top list for the server.  Val creates a *lightweight* server that waits for connections by Minecraft server lists and uses a simple protocol to get the required information.  Val is *secure*, and makes sure that all vote notifications are delivered by authentic top lists.

## Configuring VotifierAndListener

Val configures itself the first time it is run.

If you want to customize Val, simply the edit `./plugins/votifier/config.yml` file.

## Writing Vote Listeners

A vote listener implements the `VoteListener` interface which contains an implementation of the `voteMade` method.

A basic vote listener looks something like this:

    import com.vexsoftware.votifier.model.Vote;
    import com.vexsoftware.votifier.model.VoteListener;

    public class BasicVoteListener implements VoteListener {

	    public void voteMade(Vote vote) {
		    System.out.println("Received: " + vote);
	    }

    }

## Compiling Vote Listeners

Vote listeners can be compiled by including Val in the class path. For example:

	javac -cp Votifier.jar FlatfileVoteListener.java

## Encryption

Val uses one-way RSA encryption to ensure that only a trusted toplist can tell Val when a vote has been made.  When it is first run, Val will generate a 2048 bit RSA key pair and store the keys in the `./plugins/votifier/rsa` directory.  When you link Val with a toplist, the toplist will ask you for your Val public key - this is located at `./plugins/votifier/rsa/public.key` and the toplist will use this key to encrypt vote data.  It is essential that you do not share these keys with your players, as a smart player can use the key to create a spoof packet and tell Val that they voted when they really didn't.

## Protocol Documentation

This documentation is for server lists that wish to add Val support.

A connection is made to the Val server by the server list, and immediately Val will send its version in the following packet:

	"VOTIFIER <version>"

Val then expects a 256 byte RSA encrypted block (the public key should be obtained by the Val user), with the following format:

<table>
  <tr>
	<th>Type</th>
	<th>Value</th>
  </tr>
  <tr>
	<td>string</td>
	<td>VOTE</td>
  </tr>
  <tr>
	<td>string</td>
	<td>serviceName</td>
  </tr>
  <tr>
	<td>string</td>
	<td>username</td>
  </tr>
  <tr>
	<td>string</td>
	<td>address</td>
  </tr>
  <tr>
	<td>string</td>
	<td>timeStamp</td>
  </tr>
  <tr>
	<td>byte[]</td>
	<td>empty</td>
  </tr>
</table>

The first string of value "VOTE" is an opcode check to ensure that RSA was encoded and decoded properly, if this value is wrong then Val assumes that there was a problem with encryption and drops the connection. `serviceName` is the name of the top list service, `username` is the username (entered by the voter) of the person who voted, `address` is the IP address of the voter, and `timeStamp` is the time stamp of the vote.  Each string is delimited by the newline character `\n` (byte value 10).  The `space` block is the empty space that is left over, **the block must be exactly 256 bytes** regardless of how much information it holds.
