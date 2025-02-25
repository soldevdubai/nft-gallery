<template>
  <component :is="is" v-clipboard:copy="address" :class="{ aligned: verticalAlign, overflowWrap: noOwerflow }" v-if="(showTwitter && twitter) || !showTwitter">
    <template v-if="showTwitter && twitter">
      <a :href="`https://twitter.com/${twitter}`" class="pt-2" target="_blank" rel="noopener noreferrer">
        {{ twitter | toString }}
        <b-icon
          pack="fab"
          icon="twitter"
        ></b-icon>
      </a>
    </template>
    <template v-else>
      {{ name | toString }}
    </template>
  </component>
</template>

<script lang="ts">
import { Component, Prop, Watch, Mixins, Emit } from 'vue-property-decorator';
import Connector from '@vue-polkadot/vue-api';
import InlineMixin from '@/utils/mixins/inlineMixin'
import { GenericAccountId } from '@polkadot/types/generic/AccountId';
import { hexToString, isHex } from '@polkadot/util';
import { emptyObject } from '@/utils/empty';
import { Data } from '@polkadot/types';
import shortAddress from '@/utils/shortAddress';
import { get, set, update } from 'idb-keyval';
import { identityStore } from '@/utils/idbStore'
import shouldUpdate from '@/utils/shouldUpdate';

type Address = string | GenericAccountId | undefined;
type IdentityFields = Record<string, string>;

const components = {};

@Component({ components })
export default class Identity extends Mixins(InlineMixin) {
  @Prop() public address!: Address;
  @Prop(Boolean) public verticalAlign!: boolean;
  @Prop(Boolean) public noOwerflow!: boolean;
  @Prop(Boolean) public emit!: boolean;
  @Prop(Boolean) public showTwitter!: boolean;
  private identity: IdentityFields = emptyObject<IdentityFields>();

  get name(): Address {
    const name = this.identity.display;
    return name as string || shortAddress(this.resolveAddress(this.address));
  }

  get twitter(): Address {
    const twitter = this.identity.twitter;
    return twitter as string || '';
  }

  @Watch('address', { immediate: true })
  async watchAddress(newAddress: Address,  oldAddress: Address) {
    if (shouldUpdate(newAddress, oldAddress)) {
      this.identityOf(newAddress).then(id => this.identity = id);
    }
  }

  public async identityOf(account: Address): Promise<IdentityFields> {
    if (!account) {
      return Promise.resolve(emptyObject<IdentityFields>());
    }

    const address: string = this.resolveAddress(account);
    const identity = await get(address, identityStore);

    if (!identity) {
      return await this.fetchIdentity(address);
    }

    if (this.emit) {
      this.emitIdentityChange(identity);
    }

    return identity;
  }

  private handleRaw(display: Data): string {
    if (display?.isRaw) {
      return display.asRaw.toHuman() as string;
    }

    if (isHex((display as any)?.Raw)) {
      return hexToString((display as any)?.Raw);
    }

    return display?.toString();
  }

  private resolveAddress(account: Address): string {
    return account instanceof GenericAccountId ? account.toString() : account || '';
  }

  protected async fetchIdentity(address: string): Promise<IdentityFields> {
    const { api } = Connector.getInstance();

    const optionIdentity = await api?.query.identity?.identityOf(address);
    const identity = optionIdentity?.unwrapOrDefault();

    if (!identity?.size) {
      return emptyObject<IdentityFields>();
    }

    const final = Array.from(identity.info)
    .filter(([_, value]) => !Array.isArray(value) && !value.isEmpty)
    .reduce((acc, [key, value]) => {
      acc[key] = this.handleRaw(value as unknown as Data)
      return acc;
    }, {} as IdentityFields);

    update(address, () => final, identityStore);

    if (this.emit) {
      this.emitIdentityChange(final);
    }

    return final;
  }

  @Emit('change')
  emitIdentityChange(final: IdentityFields) {
    return final;
  }
}
</script>

<style scoped>
.aligned {
  vertical-align: middle;
  display: inline-block;
}

.overflowWrap {
  overflow-wrap: break-word;
}
</style>
