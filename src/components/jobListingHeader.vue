<script setup>
</script>

<template>
  <!-- Header background section -->
  <section class="container mx-auto absolute left-0 top-0 right-0 max-w-full h-[200px] bg-desaturated-dark-cyan bg-no-repeat bg-right bg-cover" style="background-image: url('src/assets/bg-header-desktop.svg');">
  </section>

  <!-- Filters section -->
  <section class="relative">
    <!-- Display active filters -->
    <div v-if="activeFilters.length" class="absolute -top-[120px] left-0 right-0 flex justify-between bg-white p-6 rounded-lg">
      <div class="flex flex-wrap gap-2">
        <!-- Render each active filter -->
        <span 
          v-for="filter in activeFilters" 
          :key="filter" 
          class="relative bg-light-grayish-cyan rounded py-1 pl-3 pr-[40px] flex items-center text-desaturated-dark-cyan font-medium"
        >
          {{ filter }}
          <!-- Remove filter button -->
          <button @click="removeFilter(filter)" class="absolute right-0 top-0 bottom-0 px-2 rounded-r-lg text-white bg-desaturated-dark-cyan text-[21px] hover:bg-darker-grayish-cyan">&times;</button>
        </span>
      </div>
      <!-- Clear all filters button -->
      <button @click="clearFilters" class="text-zinc-400 font-bold hover:underline hover:text-desaturated-dark-cyan">Clear</button>
    </div>
  </section>

  <!-- Jobs listing section -->
  <section>
    <div v-if="filteredJobs.length">
      <!-- Render each job -->
      <div 
        v-for="job in filteredJobs" 
        :key="job.id" 
        class="job flex flex-col items-start justify-between gap-5 lg:flex-row lg:items-center divide-y divide-[#919191] md:divide-[#ffffff] bg-white p-8 mb-12 md:mb-5 rounded-lg shadow-lg" 
        :class="{'border-l-4 border-desaturated-dark-cyan': job.featured}"
      >
        <div class="flex flex-wrap items-center gap-2.5 md:gap-8">
          <img class="-mt-[65px] w-[54px] sm:mt-0 sm:w-[unset]" :src="job.logo" :alt="job.company">
          <div class="flex flex-col">
            <div class="flex gap-4 mb-1.5">
              <h6 class="text-lg font-bold text-desaturated-dark-cyan">{{ job.company }}</h6>
              <div class="tags-featured flex items-center gap-2">
                <span v-if="job.new" class="bg-desaturated-dark-cyan text-white rounded-full px-2.5 py-1 text-xs font-medium leading-4">NEW!</span>
                <span v-if="job.featured" class="bg-darker-grayish-cyan text-white rounded-full px-2.5 py-1 text-xs font-medium leading-4">FEATURED</span>
              </div>
            </div>
            <h4 class="text-xl font-bold mb-1.5 text-darker-grayish-cyan cursor-pointer hover:text-desaturated-dark-cyan">{{ job.position }}</h4>
            <div class="flex items-center gap-4 text-[#707070]">
              <p>{{ job.postedAt }}</p>
              <span class="text-xs">&#x2022;</span>
              <p>{{ job.contract }}</p>
              <span class="text-xs">&#x2022;</span>
              <p>{{ job.location }}</p>
            </div>
          </div>
        </div>
        <div class="tags flex flex-wrap gap-4 pt-6 md:pt-[unset]">
          <!-- Render job tags and make them clickable to filter -->
          <div 
            class="bg-light-grayish-cyan rounded py-0.5 px-2 cursor-pointer text-desaturated-dark-cyan hover:bg-desaturated-dark-cyan hover:text-white font-semibold" 
            @click.prevent="filterByTag(job.role)"
          >
            {{ job.role }}
          </div>
          <div 
            class="bg-light-grayish-cyan rounded py-0.5 px-2 cursor-pointer text-desaturated-dark-cyan hover:bg-desaturated-dark-cyan hover:text-white font-semibold" 
            @click.prevent="filterByTag(job.level)"
          >
            {{ job.level }}
          </div>
          <div 
            class="bg-light-grayish-cyan rounded py-0.5 px-2 cursor-pointer text-desaturated-dark-cyan hover:bg-desaturated-dark-cyan hover:text-white font-semibold" 
            v-for="language in job.languages" 
            :key="language" 
            @click.prevent="filterByTag(language)"
          >
            {{ language }}
          </div>
          <div 
            class="bg-light-grayish-cyan rounded py-0.5 px-2 cursor-pointer text-desaturated-dark-cyan hover:bg-desaturated-dark-cyan hover:text-white font-semibold" 
            v-for="tool in job.tools" 
            :key="tool" 
            @click.prevent="filterByTag(tool)"
          >
            {{ tool }}
          </div>
        </div>
      </div>
    </div>
    <div v-else>
      <p>No jobs available</p>
    </div>
  </section>
</template>

<script>
export default {
  name: 'JobListingHeader',
  data() {
    return {
      jobs: [],
      filteredJobs: [],
      activeFilters: JSON.parse(localStorage.getItem('activeFilters')) || [] // Retrieve filters from local storage
    };
  },
  created() {
    // Fetch jobs data from JSON file
    fetch('./src/data.json')
      .then(response => response.json())
      .then(data => {
        this.jobs = data.sort((a, b) => b.featured - a.featured);
        // Filter jobs on load if there are active filters
        this.filteredJobs = this.activeFilters.length 
          ? this.filterJobsByActiveFilters(this.jobs) 
          : this.jobs;
      })
      .catch(error => {
        console.error('Error fetching data:', error);
      });
  },
  methods: {
    // Filter jobs based on the current active filters
    filterJobsByActiveFilters(jobs) {
      return jobs.filter(job =>
        this.activeFilters.every(filter =>
          job.role === filter || 
          job.level === filter || 
          job.languages.includes(filter) || 
          job.tools.includes(filter)
        )
      );
    },
    // Method to filter jobs based on the clicked tag
    filterByTag(tag) {
      if (!this.activeFilters.includes(tag)) {
        this.activeFilters.push(tag);
        localStorage.setItem('activeFilters', JSON.stringify(this.activeFilters)); // Save to local storage
      }
      this.filteredJobs = this.filterJobsByActiveFilters(this.jobs);
    },
    // Method to remove a filter
    removeFilter(tag) {
      this.activeFilters = this.activeFilters.filter(filter => filter !== tag); // Remove the tag from active filters
      localStorage.setItem('activeFilters', JSON.stringify(this.activeFilters)); // Update local storage
      this.filteredJobs = this.activeFilters.length 
        ? this.filterJobsByActiveFilters(this.jobs) 
        : this.jobs; // Reset to all jobs if no filters are active
    },
    // Method to clear all filters
    clearFilters() {
      this.activeFilters = [];
      localStorage.setItem('activeFilters', JSON.stringify(this.activeFilters)); // Update local storage
      this.filteredJobs = this.jobs; // Reset to all jobs
    }
  }
};
</script>
