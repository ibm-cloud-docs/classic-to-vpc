---

copyright:
  years: 2026, 2026
lastupdated: "2026-04-07"

keywords: migration, migrate, migrating, migrate infrastructure, cloud migration

subcollection: classic-to-vpc

---

{{site.data.keyword.attribute-definition-list}}

# About migration
{: #about-migration-infra}

Your decision to migrate can be driven by many factors, such as modernization, cost savings, consolidation, or closing a data center. You can also migrate to better align applications with cloud environments or to adopt new technologies like {{site.data.keyword.vpc_full}}. No matter the reason, a migration can range from something as simple as migrating a single virtual server instance to something much more complex. It can be as complex as migrating an entire application environment, pod, or even a full data center along with all its supporting components.
{: shortdesc}

![Migration approach](../images/migrate-process.svg){: caption="Migration approach" caption-side="bottom"}

## Migration approach
{: #migration-approach}

| Step | Description |
| ------ | ------------- |
| **Assess** | Do you need to migrate instances to a new data center due to data center closures? Do you want to migrate your entire {{site.data.keyword.cloud}} classic infrastructure to {{site.data.keyword.vpc_short}}? Assess your situation and identify your existing infrastructure to determine what components you have, how they are configured, and what you want to migrate. <br> Not only do you need to assess your current environment, but you need to assess the target environment to understand the capabilities, support, and differences between the two environments, if applicable. In this assessment step, you can get a general idea of the complexity of the migration so that you can develop a migration strategy. |
| **Plan** | Audit your existing infrastructure to assess which resources and components can be migrated and what dependencies exist between them. Estimate the potential impact on running services, including expected downtime and effects on connected systems, to understand the level of disruption the migration might cause. Use this to decide whether a phased or all-at-once approach makes more sense given your team's capacity and the complexity of your environment. |
| **Migrate** | After you have assessed your existing infrastructure and planned for your migration, you are ready to migrate your resources and components. Depending on your migration needs, you can choose from tools that are available to help you with the migration process. |
| **Validate** | After you migrate your resources and components into your target infrastructure, and before you make your infrastructure live, validate and test your environment to make sure that it is ready for production. This activity might also entail updating your DNS and global load balancers, routes, or retiring old services. |
{: caption="Migration approach" caption-side="top"}

## Next Steps
{: #next-steps}

* Contact your IBM Cloud Customer Success Manager (CSM) or IBM Cloud Seller for planning, migration assistance, and other queries.
* If you don’t have an assigned CSM or IBM Cloud Seller, IBM reaches out to the primary contact in the account through an email.
