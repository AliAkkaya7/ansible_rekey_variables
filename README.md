# vaulted_inventory
A collection for managing vaulted Ansible inventories. It helps to rekey all encrypted variables in the inventory repository. Can also be used to encrypt variables from clear text.

## Name
rekey_variable project for encrypted variables.

## Description

While `ansible-vault` has a rekey option to change the vault password, `ansible-vault encrypt_string` doesn't have an option. The goal of this collection is to automate changing vault passwords for all encrypted variables under the inventory group_vars and host_vars.

More details can be found in this [link](https://docs.ansible.com/ansible/latest/user_guide/vault.html#creating-encrypted-variables) on encrypted variables.

## Example for encrypted variable
----
Simply run the command below for the variable you want to encrypt, then enter the new vault password. You can use this output in your inventory to store variables as encrypted.

````bash
----
#$ ansible-vault encrypt_string '<Put the Password here>' --name 'aap_admin_password'
New Vault password:
Confirm New Vault password:
aap_admin_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          30626132326231383238356635353339313136353137333864626365636537303930303464633035
          3536646163396235623035396262663662643762333061340a313435633034373439653638396264
          36613135326136643866363039363966333164333862633335303661373033333733623361666630
          6335386436363731640a306635666261356131393031383266333361623633303064303063323835
          3433
Encryption successful
----
````

## Installation

````bash

ansible-galaxy collection install vaulted_inventory-vaulted_variable-1.0.0.tar.gz

````

## Role Variables

Role variables are defined in [defaults/main.yml](defaults/main.yml). Basically, ``new_vault_password``, ``inventory_repo_name`` should be defined to be able to change the vault password.


## Dependencies

* Ansible should be available locally.
* Inventory repositories that hold the encrypted variables should be cloned to the local directory before performing the vault password change.

## Role Tags

This role accepts the following tags to customize which part of the deployment is executed.

* aap_vault_change
* aap_post_tasks

## Example Variable File
```
my_username: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          32326637336636383161643138323532376466633661343039313733643835613930333965343638
          3331323838646363363437666638353634383431613836610a316566663564333432336561666635
          36663565646635366261353137316561623036393530346532373866643130353561623730373463
          6533393861333734340a396439323535353338623333393836623832396431373162653066386164
          6434
my_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          62623139623035303335643634356536373464353063656264353262346464656231386661343034
          3438343838373062633666396438613632393237633063660a653432633763373736343734613762
          62386338356333363061626165633138346262643239376332623065663863373166363935363838
          6534383737633732340a636234643235656263666634396138636665306663633338393439353132
          3535
my_key_passphrase: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          32623236633233646331376661633733653232356636646562663630333639366362666534333132
          3434383633393962393131326535373635393834366238330a373731656365613962643633323863
          39613036666635643664326437623534353437663031633733303666393730643064663365633236
          3133626231353536390a363439623330636130316362376638623236623563323337643664626330
          6366
```

## Example Playbook

```yaml
---
- name: Change vault password for all encrypted variables in Roles
  hosts: localhost
  gather_facts: false
  roles:
    - role: ../../rekey_variable
      when:
        - vault_change is defined
        - vault_change | bool
      tags: aap_vault_change
```

## How to run the playbook
An option such as ``--vault-id new_vault_password@prompt`` can be passed while running the playbook. It will prompt for a new vault password, and a new password can be entered. This prevents variables from being decrypted during the rerun if the playbook was interrupted during the previous run. If it is interrupted, some variables may be rekeyed with the new variables and some may remain with the old encryption key.

````bash
ansible-playbook -i <inventory_repo_path>  test.yml \
--tags "aap_vault_change" --ask-vault-pass \
    --vault-id new_vault_password@prompt
````

### How to check the variables with new key

````bash
ansible localhost -m ansible.builtin.debug -a var="encrypted_var1" -e "@<inventory_repo_path>/vault_test_data.yml" --ask-vault-pass
Vault password:
localhost | SUCCESS =>
    "encrypted_var1": "test1"
````


## Viewing encrypted variables with new vault password

It can be tested with the debug module as described in this [example](https://docs.ansible.com/ansible/latest/user_guide/vault.html#viewing-encrypted-variables).

## License
GNU General Public License (https://www.gnu.org/licenses/gpl-3.0.en.html#license-text)