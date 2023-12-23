<template>
  <div class="max-w-3xl py-12 mx-auto">
    <Card title="SHRESHTA TECHNOLOGIES">
      <form @submit.prevent="submitJournalEntry">
        <div class="space-y-10">
          <Input type="select" label="Transaction Type" :options="['Credit', 'Debit']" v-model="journalEntry.entry_type" required />
          <Input type="number" label="Amount" v-model="journalEntry.amount" required />
          <Input type="date" label="Date" v-model="journalEntry.posting_date" required />
        </div>

        <div class="mt-6">
          <Button type="submit" variant="primary" icon-left="check">Submit</Button>
        </div>
      </form>

   
    </Card>

    <ul>
      <li v-for="entry in journalEntries.data" :key="entry.name">
        {{ entry.name }} - {{ entry.title }} - {{ entry.voucher_type }} - {{ entry.company }} - {{ entry.posting_date }}
        <ul v-if="entry.accounts && entry.accounts.data">
          <li v-for="account in entry.accounts.data" :key="account.idx">
            {{ account.account }} - {{ account.party_type }} - {{ account.party }} - {{ account.debit_in_account_currency }} - {{ account.credit_in_account_currency }}
          </li>
        </ul>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue';
import { Card, Input, Button } from 'frappe-ui';
import { createListResource } from 'frappe-ui';

const journalEntries = createListResource({
  doctype: 'Journal Entry',
  fields: ["*"],
  limit: 10,
  children: {
    accounts: {
      fields: ["account", "party_type", "party", "debit_in_account_currency", "credit_in_account_currency"],
      limit: 10,
    },
  },
});
journalEntries.reload();

const journalEntry = reactive({
  company: '',
  voucher_type: 'Journal Entry',
  amount: 0,
  posting_date: '',
});

let pdfLink = null;

const submitJournalEntry = async () => {
  try {
    const response = await journalEntries.insert.submit(journalEntry);
    if (response && response.data && response.data.name) {
      generatePDFLink(response.data.name);
    }
  } catch (error) {
    console.error('Error submitting journal entry:', error);
    // Handle errors as needed
  }
};

</script>
