# tv-importofx
This repository contains a utility to download OFX files from your brokerage, convert the transactions to trade executions and import thos executions into [Tradervue](https://www.tradervue.com). 

The utility uses [py-tradervue](https://github.com/nall/py-tradervue), a Python interface to the [Tradervue API](https://github.com/tradervue/api-docs).

## Prerequisites
   * [ofxclient](https://github.com/captin411/ofxclient): A command line OFX client
   * [ofxparse](https://github.com/jseutter/ofxparse): A Python OFX parsing library
   * [keyring](https://github.com/jaraco/keyring): A Python interface to the OS-specific keyring service

Note that you currently need the development version of ofxparse. Further, if you'll be importing option transactions, you need to install a fork of that version with option support:

##### Installing py-tradervue
    FIXME

##### Installing ofxclient

    pip install ofxclient

##### Installing ofxparse (official development version)

    pip install git+https://github.com/jseutter/ofxparse

##### Installing ofxparse (fork with options support)

    pip install git+https://github.com/nall/ofxparse

## Usage
    usage: tv-importofx [-h] [--account ACCOUNT] [--file OFXFILE] [--username USERNAME]
                        [--days DAYS] [--tag TAGS] [--account_tag ACCOUNT_TAG]
                        [--allow_duplicates] [--overlay_commissions] [--debug]
                        [--debug_http]
                        {set_password,delete_password,import}
    
## Setup
### Adding OFX accounts
The [ofxclient](https://github.com/captin411/ofxclient) package already has a nice mechanism for adding OFX accounts and managing passwords, etc. Thus, the initial setup of tv-importofx requires you to run ofxclient and setup your accounts. This should be a one time step unless you change your password or want to add an additional account.

### Adding Tradervue Credentials
After setting up your OFX account, add credentials for your Tradervue account by invoking tv-importofx with the `set_password` action:

    tv-importofx set_password --username <username>

This will prompt you for the password for <username> which is stored in your OS's keyring via the Python [keyring](https://github.com/jaraco/keyring) module. You can modify this password by rerunning the command above. You can delete the key with the delete\_password action:

    tv-importofx delete_password --username <username>

Once you've setup your OFX account(s) and Tradervue credentials, you're ready to start importing executions.

## Importing
### Importing executions from OFX data
To import executions, you'll need the following:

   * Tradervue username (with valid credentials in the keyring as setup above)
   * ofxclient local\_id field for the account you're importing from. This can be found in `$HOME/ofxclient.ini`.
      * Alternately, you can give the tool a previously downloaded OFX file and use `--file` instead of `--account`.
   * Number of days of transactions to import

Once you have the above, you can import with the following command (--days defaults to 1):

    tv-importofx import --username <tradervue_username> --account <ofxclient_account_id> [--days <num_days_to_import>]

#### Additional Import Options
There are a few (rarely used?) options available in the Tradervue import API that are exposed via the command line. To enable these options, specify the appropriate command line argument as described below. For more information on these options, see the Tradervue [import API](https://github.com/tradervue/api-docs/blob/master/imports.md).

   * `--allow_duplicates`: Specify this to disable Tradervue's automatic duplicate-detection when importing 
   * `--overlay_commissions`: If this is specified, nonew trades will be created, and existing trades will be updated with commission and fee data.

## Brokerage Support
I've tested the brokerages below, though any brokerage supported by ofxclient ([ofxclient FAQ](http://captin411.github.io/ofxclient/faq.html)) should work. That said, there are sometimes issues in the OFX provided from the brokerage that need to be fixed up (for instance, OptionsXpress doesn't return the trade's execution time in GMT as it should so those dates must be fixed up before sending to Tradervue). 

   * Schwab
   * OptionsXpress

If you are able to successfully import trades from your brokerage, let me know and I'll add it to this list. 

If you're unable to download an OFX via ofxclient, but have an OFX file, you can use the `--file` option to read in a previously downloaded OFX file.

