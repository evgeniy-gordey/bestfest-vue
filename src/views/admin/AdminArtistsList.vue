<template>
  <div class="admin-artists">
    <admin-search @change="handleSearch" />
    <el-table
      :data="artistsList"
      style="width: 100%"
      @selection-change="handleSelect"
    >
      <el-table-column
        type="selection"
        width="55"
      />
      <el-table-column
        property="name"
        label="Name"
        width="120"
      />
      <el-table-column
        property="description"
        label="Description"
        width="400"
      />
      <el-table-column
        label="Country"
        width="200"
      >
        <template slot-scope="scope">
          <span>{{ formatCountry(scope.row.country) }}</span>
        </template>
      </el-table-column>
      <el-table-column
        property="genres"
        label="Genres"
      >
        <template slot-scope="scope">
          <span
            v-for="(genre, index) in scope.row.genres"
            :key="index"
          >
            {{ genre.name }},
          </span>
        </template>
      </el-table-column>
      <el-table-column width="90">
        <template slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            circle
            @click="doEdit(scope.row._id)"
          />
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            circle
            @click="doDelete(scope.row._id)"
          />
        </template>
      </el-table-column>
    </el-table>
    <div class="admin-artist__buttons">
      <div>
        <el-button
          v-if="selected.length"
          type="danger"
          @click="deleteSelected"
        >
          Delete
        </el-button>
      </div>
      <div>
        <el-button
          type="primary"
          @click="doAdd"
        >
          Add
        </el-button>
      </div>
    </div>
    <ui-pagination
      :total="artistsCount"
      :limits-list="[20, 50, 100]"
      :limit.sync="pagination.limit"
      :page.sync="pagination.page"
      @change="getPaginatedData"
    />
    <artist-modal :artist-id="editArtistId" />
  </div>
</template>

<script>
import { mapState, mapActions } from 'vuex';
import ArtistModal from '@components/admin/artist/ArtistModal';
import { continents, countries } from 'countries-list';
import AdminSearch from '@views/admin/AdminSearch';
import { debounce } from 'lodash';

export default {
  name: 'AdminArtists',
  components: { AdminSearch, ArtistModal },
  data() {
    return {
      editArtistId: null,
      selected: [],
      search: '',
      pagination: {
        page: 1,
        limit: 20,
      },
      debouncedGetData: () => {}
    };
  },
  computed: {
    ...mapState('admin/artist', ['artistsList', 'artistsCount']),
    countries() {
      return countries;
    }
  },
  created() {
    this.getPaginatedData();
    this.debouncedGetData = debounce(() => {
      this.getPaginatedData();
    }, 400);
  },
  methods: {
    ...mapActions('admin/artist', ['a_getArtistsList', 'a_createArtist', 'a_updateArtist', 'a_deleteArtists']),
    ...mapActions('modals', ['a_openModal']),
    handleSearch(search) {
      this.search = search;
      this.debouncedGetData();
    },
    getPaginatedData() {
      const params = {
        page: this.pagination.page,
        limit: this.pagination.limit,
        search: this.search
      };
      this.a_getArtistsList(params);
    },
    handleSelect(value) {
      this.selected = value;
    },
    deleteSelected() {
      const payload = {
        ids: this.selected.map(item => item._id)
      };
      this.a_deleteArtists(payload);
    },
    doAdd() {
      this.editArtistId = null;
      this.a_openModal('artistCRUD');
    },
    doEdit(artistId) {
      this.editArtistId = artistId;
      this.a_openModal('artistCRUD');
    },
    doDelete(id) {
      const payload = {
        ids: [id]
      };
      this.a_deleteArtists(payload);
    },
    formatCountry(code) {
      const country = countries[code];
      return `${continents[country.continent]}, ${country.name} ${country.emoji}`;
    }
  }
};
</script>

<style scoped lang="scss">
.admin-artist {
  &__buttons {
    display: flex;
    justify-content: space-between;
    margin: 20px 0;
  }
}

</style>
