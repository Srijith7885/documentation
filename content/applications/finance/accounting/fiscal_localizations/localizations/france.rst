======
France
======

FEC
===

A FEC file (Fichier des Ecritures Comptables) contains all the accounting data and entries recorded in all 
the accounting journals for a financial year. The entries in the file must be arranged in chronological order.

Since 1 January 2014, every French company is required to produce and transmit this file upon request 
by the tax authorities for audit purposes.

Import 
------

In order to make the onboarding of new users easier, Odoo Enterprise's French fiscal localization
includes access to the ``account_fec_import`` module, that enables the import of existing FEC files
from older software.

.. tip::
    | To do so, activate the Developer mode and then, in the menu, you can find the item filed under:
    |
    | :menuselection:`Accounting --> Configuration --> FEC Import`

A modal window will then appear, to make the user select the FEC file to be imported.
The import procedure will start right after pressing the "Import" button.

.. note::
    It will take no particular action or computation when importing FEC files from different years,
    so should two files both contain any "Reports à Nouveaux" (RAN) with the starting balance of the year,
    you might need to cancel those entries in the User Interface. 
    Indeed, Odoo's logic makes those entries (RAN) useless.


File formats
~~~~~~~~~~~~

FEC files can be either in CSV or XML format, the uploaded file extension is used to determine the format.

Our module expects both file formats to meet the following technical specifications:

* **Encoding** : preferably UTF-8. Though, ASCII or Latin-1 are also supported.
* **Separator** : any of these: `;` or `|` or `,` or `TAB`.
* **Line terminators**: both CR+LF (`\\r\\n`) and LF (`\\n`) character groups are supported.
* **Date format** : `%Y%m%d`

CSV
***

The FEC CSV file has a plain text format representing a data table, with the first line being a header
and defining the list of fields for each entry, and each following line representing one accounting entry,
in no predetermined order.

* **File Extension**: any but `.xml`.

XML
***

The FEC XML file is more structured than its CSV counterpart, with nested tags representing:

*   | The **whole file**  `<comptabilite/>`
    | Root tag.

*   | The **company** `<exercice/>`
    | Contains the tag 'DateCloture' which is ignored.

*   | The **journal** `<journal/>`
    | Contains the tags: ['JournalCode', 'JournalLib']

*   | The **move** `<ecriture/>`
    | Contains the tags: ['EcritureNum', 'EcritureDate', 'EcritureLib', 'PieceRef', 'PieceDate', 'EcritureLet', 'DateLet', 'ValidDate']

*   | The **lines** `<ligne/>`
    | Contains the tags ['CompteNum', 'CompteLib', 'CompteAuxNum', 'CompAuxLib', 'CompteAuxLib', 'MontantDevise', 'Idevise', 'Debit', 'Credit']

* **File Extension**: `.xml`.
* **XSD Schema**: `a47a-i-vii-1.xsd <https://www.impots.gouv.fr/portail/files/media/3_Documentation/guides_notices/fichiers_XSD/a47a-i-vii-1.xsd>`_


Fields description and use
~~~~~~~~~~~~~~~~~~~~~~~~~~

+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
|  # | Field name    | Description                                                                    | Use                                                         | Format               |
+====+===============+================================================================================+=============================================================+======================+
| 01 | JournalCode   | Journal Code                                                                   | journal.code and journal.name if JournalLib is not provided | Alphanumeric         |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 02 | JournalLib    | Journal Label                                                                  | journal.name                                                | Alphanumeric         |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 03 | EcritureNum   | Numbering specific to each journal /  sequence number of the accounting entry  | move.name                                                   | Alphanumeric         |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 04 | EcritureDate  | Accounting entry Date                                                          | move.date                                                   | Date (yyyyMMdd)      |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 05 | CompteNum     | Account Number                                                                 | account.code                                                | Alphanumeric         |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 06 | CompteLib     | Account Label                                                                  | account.name                                                | Alphanumeric         |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 07 | CompAuxNum    | Secondary account Number (accepts null)                                        | partner.ref                                                 | Alphanumeric         |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 08 | CompAuxLib    | Secondary account Label (accepts null)                                         | partner.name                                                | Alphanumeric         |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 09 | PieceRef      | Document Reference                                                             | move.ref and move.name if EcritureNum is not provided       | Alphanumeric         |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 10 | PieceDate     | Document Date                                                                  | move.date                                                   | Date (yyyyMMdd)      |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 11 | EcritureLib   | Account entry Label                                                            | move_line.name                                              | Alphanumeric         |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 12 | Debit         | Debit amount                                                                   | move_line.debit                                             | Float                |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 13 | Credit        | Credit amount (Field name "Crédit" is not allowed)                             | move_line.credit                                            | Float                |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 14 | EcritureLet   | Accounting entry cross reference (accepts null)                                | move_line.fec_matching_number                               | Alphanumeric         |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 15 | DateLet       | Accounting entry date (accepts null)                                           | unused                                                      | Date (yyyyMMdd)      |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 16 | ValidDate     | Accounting entry validation date                                               | unused                                                      | Date (yyyyMMdd)      |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 17 | Montantdevise | Currency amount (accepts null)                                                 | move_line.amount_currency                                   | Float                |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+
| 18 | Idevise       | Currency identifier (accepts null)                                             | currency.name                                               | Alphanumeric         |
+----+---------------+--------------------------------------------------------------------------------+-------------------------------------------------------------+----------------------+

These two fields can be found in place of the others in the sequence above.

+----+---------------+--------------------------------------------------------------------------------+-----------------------------------------------------+----------------------+
| 12 | Montant       | Amount                                                                         | move_line.debit or move_line.credit                 | Float                |
+----+---------------+--------------------------------------------------------------------------------+-----------------------------------------------------+----------------------+
| 13 | Sens          | Can be "C" for Credit and "D" for Debit                                        | determines move_line.debit or move_line.credit      | Char                 |
+----+---------------+--------------------------------------------------------------------------------+-----------------------------------------------------+----------------------+


Implementation details
~~~~~~~~~~~~~~~~~~~~~~

The following accounting entities  will be imported from the FEC files: **Accounts, Journals, Partners** and **Moves**.

Our module will determine the encoding, the line-terminator character and the separator that are used in the file.

A check then is performed to see if every line has the correct number of field, corresponding to the header.

If the check passes, then the file is read in full, kept in memory, and scanned. 
Accounting entities will be imported one type at a time, in the following order.


Accounts
********

Every accounting entry is related to an account, which should be determined by the field ``CompteNum``

Code matching
*************

Should a similar account code already be present in the system, the existing one will be used
instead of creating a new one.

Accounts in Odoo generally have a number of digits that is default for the fiscal localization.
As the FEC module is related to the French localization, the default number of relevant digits is 6.

This means that the account codes will be right-trimming the trailing zeroes, and that the comparison
between the account codes in the FEC file and the ones already existing in Odoo will be performed
only on the first six digits of the codes.

**Example**: the account code 65800000 in the file will be matched against an existing 658000
account in Odoo, and that will be used instead of creating a new one.

Reconcilable flag
*****************

An account is technically flagged as **reconcilable** if the first line in which it appears has the "EcritureLet" 
field filled in, as this flag means that the accounting entry is going to be reconciled with another one.

.. note::
    In case the line somehow has this field not filled in, but the entry still had to be reconciled
    with a payment that yet been recorded, this won't be a problem anyway; the account will be flagged
    as reconcilable as soon as the import of the move lines will require it.

Account type and Templates matching
***********************************

As the **type** of the account is not specified in the FEC format, **new** accounts will be created
with the default type 'Current Assets' and then, at the end of the import process, they will be
matched against the installed Chart of Account templates.
Also the **reconcile** flag will also be computed this way.

The match is done with the left-most digits, starting by using all digits, then 3, then 2.

Example: ::

    Template:  400000: Fournisseurs et comptes rattachés
    CompteNum: 40100000
               ^^

The type of the account will then be flagged as '**payable**' and **reconcilable** as per the account template.

Journals
********

Journals are also checked against those already existing in Odoo to avoid duplicates,
also in the case of multiple FEC files imports.

Should a similar journal code already be present in the system, the existing one will be used
instead of creating a new one.

New journals will have their name prefixed by the string ``FEC-``

e.g. `ACHATS --> FEC-ACHATS`

The journals will **not** be archived, the user will be entitled to handle them as he wishes.

Journal type determination
**************************

The journal type is also not specified in the format (as per the accounts) and therefore it is
at first created with the default type "general".

At the end of the import process, the type is determined as per these rules regarding related moves and accounts:

* | **bank** : Moves in these journals will always have a line (debit or credit) impacting a liquidity account.
  | ('cash' / 'bank' can be interchanged, so 'bank' is set everywhere when this condition is met)
* | **sale** : Moves in these journals will mostly have debit lines on receivable accounts and credit lines  on tax income accounts.
  | Sale refund journal items will be debit/credit inverted.
* | **purchase** : Moves in these journals will mostly have credit lines on payable accounts and debit lines on expense accounts.
  | Purchase refund journal items will be debit/credit inverted.
* | **general** : for everything else.

.. note::
    A minimum of 3 moves is necessary for journal type identification.
    A threshold of 70% of moves must correspond to a criteria for a journal_type to be determined.

**Example**::

    Journal id = 5
    Moves:
        has a sale account line and no purchase account line = 0     ratio = 0
        has a purchase account line and no sale account line = 1     ratio = 0.25
        has a liquidity account line                         = 3     ratio = 0.75
                                                        ----------
                                                        Total: 4

The journal type is "bank", because the bank moves ratio 3/4 (0.75) exceeds the threshold (0.7)

Partners
********

Each partner keeps its Reference from the field "CompAuxNum", which will be searchable from Odoo,
in line with former FEC imports on the accounting expert's side for fiscal/audit purposes.

Users can merge partners with the Data Cleaning App, where Vendors and Customers or similar
partner entries may be merged by the user, with assistance from the system that will group them
by similar entries.

Moves
*****

Entries will be immediately posted and reconciled after submission, using the "EcritureLet" field
to do the matching between the entries themselves.

The "EcritureNum" field represents the name of the moves. We noticed that sometimes it may be not be filled in.
In this case, the field "PieceRef" is used.

Rounding issues
***************

There is a rounding tolerance with a currency-related precision on debit and credit *(i.e. 0.01 for EUR)*
Under this tolerance, a new line will be added to the move, named 'Import rounding difference',
targeting the accounts:

* Charges diverses de gestion courante (658000) for added debits
* Produits diverses de gestion courante (758000) for added credits

Missing move name
*****************

Should the "EcritureNum" not be filled in, it may also happen that the "PieceRef" field is also
not suited to determine the move name (it may be used as an accounting move line reference) leaving
no way to actually find which lines are to be grouped in a single move, and effectively
impeding the creation of balanced moves.

One last attempt is made, grouping all lines from the same journal and date ("JournalLib", "EcritureDate").
Should this grouping generate balanced moves (sum(credit) - sum(debit) = 0), then each different
combination of journal and date will create a new move.

e.g. ACH + 2021/05/01 ---> new move on journal ACH with name '20210501'.

Should this attempt fail, the user will be prompted an error message with all the move lines
that are supposedly unbalanced.

Partner information
*******************

If a line has the partner information specified, the information is copied to the accounting Move
itself if the targeted Journal is of type "payable" or "receivable".

Export
------

| If you have installed the French Accounting, you will be able to download the FEC.
| For this, go in :menuselection:`Accounting --> Reporting --> France --> FEC`.

.. tip::
    If you do not see the submenu **FEC**, go in **Apps** and search for the module
    called **France-FEC** and verify if it is well installed.

More Information
----------------

You will find more information about the FEC format here:

* `Official Technical Specification (fr) <https://www.legifrance.gouv.fr/codes/article_lc/LEGIARTI000027804775>`_
* `Official FEC Testing tool (last updated in 2018) <https://github.com/DGFiP/Test-Compta-Demat>`_

French Accounting Reports
=========================

If you have installed the French Accounting, you will have access to some accounting reports specific to France:

- Bilan comptable
- Compte de résultats
- Plan de Taxes France

Get the VAT anti-fraud certification with Odoo
==============================================

As of January 1st 2018, a new anti-fraud legislation comes into effect
in France and DOM-TOM. This new legislation stipulates certain criteria
concerning the inalterability, security, storage and archiving of sales data.
These legal requirements are implemented in Odoo, version 9 onward,
through a module and a certificate of conformity to download.

Is my company required to use an anti-fraud software?
-----------------------------------------------------

Your company is required to use an anti-fraud cash register software like
Odoo (CGI art. 286, I. 3° bis) if:

* You are taxable (not VAT exempt) in France or any DOM-TOM,
* Some of your customers are private individuals (B2C).

This rule applies to any company size. Auto-entrepreneurs are exempted from
VAT and therefore are not affected.

Get certified with Odoo
-----------------------

Getting compliant with Odoo is very easy.

Your company is requested by the tax administration to deliver a certificate
of conformity testifying that your software complies with the anti-fraud 
legislation. This certificate is granted by Odoo SA to Odoo Enterprise users
`here <https://www.odoo.com/my/contract/french-certification/>`_.
If you use Odoo Community, you should
:doc:`upgrade to Odoo Enterprise </administration/enterprise>`
or contact your Odoo service provider.

In case of non-conformity, your company risks a fine of €7,500.

To get the certification just follow the following steps:

* Install the anti-fraud module fitting your Odoo environment from the
  *Apps* menu:

  * if you use Odoo Point of Sale: *l10n_fr_pos_cert*: France - VAT Anti-Fraud Certification for Point of Sale (CGI 286 I-3 bis)
  * in any other case: *l10n_fr_certification*: France - VAT Anti-Fraud Certification (CGI 286 I-3 bis)

* Make sure a country is set on your company, otherwise your entries won’t be
  encrypted for the inalterability check. To edit your company’s data,
  go to :menuselection:`Settings --> Users & Companies --> Companies`.
  Select a country from the list; Do not create a new country.
* Download the mandatory certificate of conformity delivered by Odoo SA `here <https://www.odoo.com/my/contract/french-certification/>`__.

.. note::
   * To install the module in any system created before
     December 18th 2017, you should update the modules list.
     To do so, activate the :doc:`Developer mode </applications/general/developer_mode>`.
     Then go to the *Apps* menu and press *Update Modules List* in the top-menu.
   * In case you run Odoo on-premise, you need to update your installation
     and restart your server beforehand.
   * If you have installed the initial version of the anti-fraud module
     (prior to December 18th 2017), you need to update it.
     The module's name was *France - Accounting - Certified CGI 286 I-3 bis*.
     After an update of the modules list, search for
     the updated module in *Apps*, select it and click *Upgrade*.
     Finally, make sure the following module *l10n_fr_sale_closing*
     is installed.

Anti-fraud features
-------------------

The anti-fraud module introduces the following features:

* **Inalterability**: deactivation of all the ways to cancel or modify
  key data of POS orders, invoices and journal entries;
* **Security**: chaining algorithm to verify the inalterability;
* **Storage**: automatic sales closings with computation of both period
  and cumulative totals (daily, monthly, annually).

Inalterability
~~~~~~~~~~~~~~

All the possible ways to cancel and modify key data of paid POS orders,
confirmed invoices and journal entries are deactivated,
if the company is located in France or in any DOM-TOM.

.. note:: If you run a multi-companies environment, only the documents of
 such companies are impacted.

Security
~~~~~~~~

To ensure the inalterability, every order or journal entry is encrypted
upon validation.
This number (or hash) is calculated from the key data of the document as
well as from the hash of the precedent documents.

The module introduces an interface to test the data inalterability.
If any information is modified on a document after its validation,
the test will fail. The algorithm recomputes all the hashes and compares them
against the initial ones. In case of failure, the system points out the first
corrupted document recorded in the system.

Users with *Manager* access rights can launch the inalterability check.
For POS orders, go to
:menuselection:`Point of Sales --> Reporting --> French Statements`.
For invoices or journal entries,
go to :menuselection:`Invoicing/Accounting --> Reporting --> French Statements`.

Storage
~~~~~~~

The system also processes automatic sales closings on a daily, monthly
and annual basis.
Such closings distinctly compute the sales total of the period as well as
the cumulative grand totals from the very first sales entry recorded
in the system.

Closings can be found in the *French Statements* menu of Point of Sale,
Invoicing and Accounting apps.

.. note::
 * Closings compute the totals for journal entries of sales journals (Journal Type = Sales).

 * For multi-companies environments, such closings are performed by company.

 * POS orders are posted as journal entries at the closing of the POS session.
   Closing a POS session can be done anytime.
   To prompt users to do it on a daily basis, the module prevents from resuming
   a session opened more than 24 hours ago.
   Such a session must be closed before selling again.

 * A period’s total is computed from all the journal entries posted after the
   previous closing of the same type, regardless of their posting date.
   If you record a new sales transaction for a period already closed,
   it will be counted in the very next closing.

.. tip:: For test & audit purposes such closings can be manually generated in the
   :doc:`Developer mode </applications/general/developer_mode>`. Then go to
   :menuselection:`Settings --> Technical --> Automation --> Scheduled Actions`.


Responsibilities
----------------

Do not uninstall the module! If you do so, the hashes will be reset and none
of your past data will be longer guaranteed as being inalterable.

Users remain responsible for their Odoo instance and must use it with
due diligence. It is not permitted to modify the source code which guarantees
the inalterability of data.

Odoo absolves itself of all and any responsibility in case of changes
in the module’s functions caused by 3rd party applications not certified by Odoo.


More Information
----------------

You will find more information about this legislation in the official documents:

* `Frequently Asked Questions <https://www.economie.gouv.fr/files/files/directions_services/dgfip/controle_fiscal/actualites_reponses/logiciels_de_caisse.pdf>`_
* `Official Statement <http://bofip.impots.gouv.fr/bofip/10691-PGP.html?identifiant=BOI-TVA-DECLA-30-10-30-20160803>`_
* `Item 88 of Finance Law 2016 <https://www.legifrance.gouv.fr/affichTexteArticle.do?idArticle=JORFARTI000031732968&categorieLien=id&cidTexte=JORFTEXT000031732865>`_
