# hook_civicrm_generateIdentifier

## Summary

allows you to generate custom external identifiers (for contacts, invoices and campaigns).

## Availability

This hook is available in CiviCRM 4.7.27+ (maybe depends on the commit).

## Definition

     hook_civicrm_generateIdentifier(&$identifier, $context, $object) 

## Parameters

-   @param string $identifier - can be enhanced by the hook
-   @param string $context - the different context are listed below.
-   @param object reference to the object the identifier is stored

## Details

On a number of places CiviCRM generates numbers that have meaning in the outside world. An example is the invoice number of a 
contribution. Sometimes an organization wants to define its own format for these indentifiers. At the moment the following identifiers are covered (denote them in the $context parameter).

- contact_external_identifier: When new contact is created.
- invoice_number: when the pdf of the invoice is generated.
- creditnote_id: when a credit not is generated.


## Example

     function customidentifiers_civicrm_generateIdentifier(&$identifier, $context, $object) {
       // just a way to generate a random number.
       $uniqueNr = (new DateTime('now'))->format('YmdHis');
       switch ($context) {
         case 'contact_external_identifier':
           $identifier = 'external_' . $uniqueNr;
           break;
     
         case 'creditnote_id':
           $identifier = 'cn' . $uniqueNr;
           break;
     
         case 'invoice_number':
           // only generate a invoice_number if it is not already there
           if (empty($object->invoice_number)) {
     
             $prefix = 'INVPREV';
             $counter_position = strlen($prefix) + 1;
     
             /* calculate a sequence */
             $last_id = CRM_Core_DAO::singleValueQuery("
             SELECT MAX(CAST(SUBSTRING(`invoice_id` FROM {$counter_position}) AS UNSIGNED))
             FROM `civicrm_contribution`
             WHERE `invoice_id` REGEXP '{$prefix}[0-9]+$';");
     
             if ($last_id) {
               $identifier = $prefix . ($last_id + 1);
             }
             else {
               $identifier = "{$prefix}1";
             }
           }
           else {
             // return the identifier that is already part
             // of the contribution, otherwise it is
             // replaced by the generated standard one
             $identifier = $object->invoice_number;
           }
       }
     }

