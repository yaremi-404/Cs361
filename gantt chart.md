## Pizza Ordering System
_Yaremi Dominguez_
_November 4, 2026_
_In class chalenge 6.3_

``` mermaid
gantt
    title Pizza Ordering System
    dateFormat YYYY-MM-DD

    section Planning
        Project Management       :done, planning, 2026-03-01, 20d
        Requirements and Design  :done, reqs, after planning, 30d
    section Build
        Customer Web App    :active, proto, after reqs, 70d
        Back-end            :active, services, after proto, 40d
        Staff               :active , tools,  after services, 14d
    section Launch
        Testing          :testing, after tools, 16d
        Launch          :active, marketing, after testing, 10d

```