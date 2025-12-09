---
title: "Week 8 Worklog"

weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---



### Week 8 Objectives:

* Integrate map display and geocoding for room locations.
* Complete search and filtering functionality.
* Test all major features of the system.
* Prepare Proposal Review if required.

### Tasks for this Week:
| Day | Tasks                                                                                                                                                  | Start Date  | Completion Date | References                                 |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------- | ---------------- | ------------------------------------------- |
| 2   | - Backend: Integrate **Vietmap Geocoding API** into `roomCrud.js` to store coordinates <br> - Frontend: Add **Vietmap-gl** to `RoomMap.js`              | 17/11/2025  | 17/11/2025       | https://maps.vietmap.vn/                    |
| 3   | - Backend: Complete filtering logic (price, district, distance) in `searchRooms.js`                                                                     | 18/11/2025  | 18/11/2025       | https://docs.aws.amazon.com/amazondynamodb/ |
| 4   | - Frontend: Build **Search Page (`search/page.js`)** with filters <br> - Display results on **map + list**                                             | 19/11/2025  | 19/11/2025       | https://nextjs.org/docs                     |
| 5   | - Manual Testing: posting rooms (with coordinates), searching, filtering                                                                               | 20/11/2025  | 20/11/2025       |                                             |
| 6   | - Prepare **Proposal Review**: summarize completed features & presentation materials                                                                    | 21/11/2025  | 21/11/2025       |                                             |

### Week 8 Achievements:

* Successfully integrated Vietmap Geocoding:
  * Auto-fetch coordinates when users post a room  
  * Store coordinates in DynamoDB via Lambda
* Frontend map display using **Vietmap-gl** works smoothly.
* Completed search and filtering:
  * Price filter  
  * District filter  
  * Distance-based filter  
  * Improved search logic in Lambda
* Search Page completed:
  * UI filters  
  * Room list  
  * Map with markers
* Testing results:
  * Posting → geocoding → storage works correctly  
  * Searching & filtering functions correctly  
  * UI + API synced across modules
* **Final Result:** Search + Map modules are fully functional — the core of the app is now complete.
