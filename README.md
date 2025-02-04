module.exports = {
  docs: [
    'introduction',
    `image-lifecycle-management`,
    {
      type: 'category',
      label: 'EKS Clusters',
      items: [
        'platforms/eks/overview',
        'platforms/eks/architecture',
        'platforms/eks/quick-start',
        {
          type: 'category',
          label: 'Guides',
          items: [
            'platforms/eks/guides/onboarding',
            'platforms/eks/guides/cluster-creation',
            'platforms/eks/guides/cluster-life-cycle',
            'platforms/eks/guides/cicd-node-creation',
            'platforms/eks/guides/app-deployment',
            'platforms/eks/guides/migration-automation-util',
            'platforms/eks/guides/cluster-observability',
            'platforms/eks/guides/amp-amg',
            'platforms/eks/guides/cluster-cost',
            'platforms/eks/guides/troubleshoot-eks',
            'platforms/eks/guides/httpd-forgerock',
            'platforms/eks/guides/kubecost-onboarding',
            'platforms/eks/guides/selfheal-guide',
            'platforms/eks/guides/istio-userguide',
            'platforms/eks/guides/efs',
            'platforms/eks/guides/npd',
            'platforms/eks/guides/observability-nr',

          ]
        },
        {
          type: 'category',
          label: 'Runbooks',
          items: [
            'platforms/eks/guides/runbook-app',
            'platforms/eks/guides/runbook-admin',
            'platforms/eks/guides/important-jenkins-job'
          ]
        },
        'platforms/eks/policies',
        'platforms/eks/support',
        'platforms/eks/currentIssues/ongoing-issues',
        {
          type: 'category',
          label: 'Releases',
          items: [
            'platforms/eks/releases/release',
            'platforms/eks/releases/bosun-2',
            'platforms/eks/releases/bosun-3',
            'platforms/eks/releases/bosun-4',
            'platforms/eks/releases/bosun-5',
            'platforms/eks/releases/bosun-6',
            'platforms/eks/releases/bosun-7',
            'platforms/eks/releases/bosun-8',
            'platforms/eks/releases/bosun-9',
            'platforms/eks/releases/gpu-userguide'
          ]
        }
      ]
    },
    {
      type: 'category',
      label: 'GKE Clusters',
      items: [
        'platforms/gke/overview',
        'platforms/gke/architecture',
        {
          type: 'category',
          label: 'Cluster Components',
          items: [
            'platforms/gke/architecture/networking',
            'platforms/gke/architecture/storage',
            'platforms/gke/architecture/autoscaling',
            'platforms/gke/architecture/security',
            'platforms/gke/architecture/observability',
            'platforms/gke/architecture/configuration',
          ]
        },
        {
          type: 'category',
          label: 'Guides',
          items: [
            'platforms/gke/guides/onboarding',
            'platforms/gke/guides/manage-cluster',
            'platforms/gke/guides/app-deployment',
            'platforms/gke/guides/container-security',
            'platforms/gke/guides/application-cicd',
            'platforms/gke/guides/anthos-service-mesh',
            'platforms/gke/guides/cluster-monitoring',
            'platforms/gke/guides/cluster-logging',
            'platforms/gke/guides/policy-enforcement',
            {
              type: 'category',
              label: 'Autoscaling',
              items: [
                'platforms/gke/guides/autoscaling/hpa',
                'platforms/gke/guides/autoscaling/vpa'
              ]
             },
          ]
        },
        {
          type: 'category',
          label: 'Releases',
          items: [
            'platforms/gke/releases/release-schedule',
            'platforms/gke/releases/release-notes'
          ]
        },
            
        'platforms/gke/support'
      ]
    },
    {
      type: 'category',
      label: 'Openshift Clusters',
      items: [
        'platforms/ocp/overview',
        'platforms/ocp/getting-started',
        'platforms/ocp/cluster-info',
        'platforms/ocp/new-features',
        'platforms/ocp/architecture',
        'platforms/ocp/quick-start',
        {
          type: 'category',
          label: 'Guides',
          items: ['platforms/ocp/guides/best-practices', 'platforms/ocp/guides/deploy-app', 'platforms/ocp/guides/commands']
        },
        'platforms/ocp/security',
        'platforms/ocp/support',
        'platforms/ocp/faq'
      ]
    },
    {
      type: 'category',
      label: 'OKE Clusters',
      items: [
        'platforms/oke/overview',
        'platforms/oke/architecture',
        'platforms/oke/quick-start',
        {
          type: 'category',
          label: 'Cluster Components',
          items: [
            'platforms/oke/architecture/networking',
            'platforms/oke/architecture/storage',
            'platforms/oke/architecture/autoscaling',
            'platforms/oke/architecture/security',
            'platforms/oke/architecture/observability',
            'platforms/oke/architecture/platform-addons'
          ]
        },
             
        {
          type: 'category',
          label: 'Guides',
          items: [
            'platforms/oke/guides/onboarding',
            'platforms/oke/guides/manage-cluster',
            'platforms/oke/guides/container-security',
            'platforms/oke/guides/expose-app',
            'platforms/oke/guides/storage',
            {
              type: 'category',
              label: 'Observability',
              items: [
                'platforms/oke/guides/monitoring/monitoring',
                'platforms/oke/guides/monitoring/logging'
              ]
            },
            {
              type: 'category',
              label: 'Autoscaling',
              items: [
                'platforms/oke/guides/autoscaling/hpa',
                'platforms/oke/guides/autoscaling/vpa',
                'platforms/oke/guides/autoscaling/ca'
              ]
            },
            'platforms/oke/guides/gitlab-runner'
          ]
        },
        'platforms/oke/support',
        {
          type: 'category',
          label: 'Releases',
          items: ['platforms/oke/releases/release-schedule', 'platforms/oke/releases/release-notes']
        }
      ]
    },
    {
      type: 'category',
      label: 'Legacy Clusters',
      items: [
        'platforms/on-prem/overview',
        'platforms/on-prem/architecture',
        'platforms/on-prem/quick-start',
        {
          type: 'category',
          label: 'Guides',
          items: [
            'platforms/on-prem/guides/deploy-app',
            'platforms/on-prem/guides/k8s-devx-deployment',
            'platforms/on-prem/guides/k8s-nonprod-deployment',
            'platforms/on-prem/guides/k8s-prod-deployment',
            'platforms/on-prem/guides/k8s-cicd',
            'platforms/on-prem/guides/cost',
            'platforms/on-prem/guides/legacy-firewall',
            'platforms/on-prem/guides/logging',
            'platforms/on-prem/guides/monitoring',
            'platforms/on-prem/guides/httpd-siteminder',
            'platforms/on-prem/guides/shared-vips',
            'platforms/on-prem/guides/hpa-overview',
            'platforms/on-prem/guides/secret-management',
            'platforms/on-prem/guides/httpd-forgerock',
            'platforms/on-prem/guides/traefik-v2',
            'platforms/on-prem/guides/upgrade-reqs-119',
            'platforms/on-prem/guides/jumpserver'
          ]
        },
        'platforms/on-prem/policies',
        'platforms/on-prem/patching-schedule',
        'platforms/on-prem/support'
      ]
    },
    {
      type: 'category',
      label: 'Global Guides',
      items: [
        'guides/developer-guide',
        'guides/onboarding-flow',
        'guides/devx-setup',
        'guides/artifactory-guide',
        'guides/docker-overview',
        'guides/sysdig-overview',
        'guides/helm-integration',
        'guides/best-practice-docker-file',
        'guides/best-practice-deployment-yaml',
        'guides/common-problems'
      ]
    },
    'policies',
    'support',
    'contributing'
  ]
};
