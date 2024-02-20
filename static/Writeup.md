# Project 3 Writeup for Team 49ers

## Rationale for your design decisions

*How did you choose your particular visual encodings and interaction techniques?*

    In designing the visual encodings and interaction techniques for this data visualization, several considerations were taken into account. The choice of layered circle encoding with line encoding allows for clear representation of data points and trends. Axes are included to provide context and aid in interpretation, with the scale dynamically adjusting based on the highest value of the selected energy sources. The y-axis stays grounded at 0 as the base for consistency and to make trends and comparisons easier for the user. The x-axis formatting ensures readability by removing commas and labels centered for clarity. A tooltip feature is implemented to provide additional information, displaying the year and corresponding value upon interaction. Background color changes correspond to the energy source being hovered over, enhancing user understanding. The suggestion button facilitates user interaction by plotting selected countries and removing suggestion text upon click, while also reappearing when changes are made to the search bar. A clear button resets the visualization to default settings, with the United States set as the default country. The legend offers further customization, allowing users to select energy sources to display on the chart, with background colors reflecting the selected source for clarity. When deselected, energy sources have no background color to avoid clutter. These design choices aim to optimize user experience and facilitate effective data exploration and analysis.

*What alternatives did you consider and how did you arrive at your ultimate choices?*

    In the process of designing our visualization, we considered several alternatives before arriving at our ultimate choices. One design choice we made was showing one country at a time. Initially, we had wanted to compare multiple countries at a time, but as we saw the number of features we were already encoding with color, we thought the graph would become cluttered if we tried to add in another country. We considered alternatives like using the same color source for the features, and then changing the shape of the lines and circles that encode the different countries, but even with these changes the graph easily became too cluttered to read. We ended up deciding that simply allowing a person to look up a country and see its energy source composition was enough utility that we could focus on optimizing the display of one country at a time.
    Regarding the legend, we deliberated between a simple static legend and an interactive one that allows users to toggle features on and off. Given the multitude of features available for exploration, we recognized the importance of enabling users to customize their view based on their specific interests. Thus, the interactive legend was chosen to enhance user control and focus.
    Another idea considered was incorporating an interactive globe for visualization. However, after careful consideration, we determined that providing textual encoding indicating the highest energy source would offer more immediate and actionable insights to users. This decision prioritized clarity and ease of interpretation for the user interface.
    By carefully weighing these alternatives, we arrived at design choices that prioritize user experience, clarity, and utility in exploring energy source data.

## An overview of your development process. Include a commentary on the development process, including answers to the following questions

*Describe how the work was split among the team members.*

    The team's work on the project was organized through a collaborative approach with regular discussions and division of tasks based on expertise and interest. Initially, the team outlined the conceptual framework of the project, drawing inspiration from previous labs (4 and 5) and demos. During daily meetings, team members tackled different aspects of the project, addressing issues collaboratively as they arose.
    Work was divided among team members according to specific components of the visualization. Tasks included handling data loading and cleaning using Pandas, plotting points and lines, implementing the search bar functionality, and developing country suggestions based on user input. Additionally, features such as the "clear" button for the search bar and the inclusion of line segments between data points to highlight trends were incorporated.
    As the project progressed, the team iteratively refined the visualization, adding functionality such as toggle buttons to include or exclude different energy sources and adjusting the y-axis scaling to accommodate changes in the displayed data. Feedback from a TA prompted the team to reassess the visualization's effectiveness in conveying the significance of the data. Consequently, the decision was made to replace the initially proposed interactive globe with a text encoding approach to enhance user understanding.
    Overall, the team's collaboration and division of labor allowed for the successful development and improvement of the visualization, resulting in a more informative and user-friendly final product.


### Hours Spent

| Date | Meeting Time | Individual Work | Total Hours Spent  |
|------|--------------|-----------------|--------------------| 
| Thursday 2/8   | 7:30pm - 9:30pm (2 x 3 = 6)     | 0 | 6   | 
| Sunday 2/11    | 8:30pm - 10pm (1.5 x3 = 4.5) | 0 | 4.5 |
| Monday 2/12    |                              | 3 each | 6 |
| Tuesday 2/13   | 9am-11:50, 12pm - 1:30pm (4.5 x3 = 13.5) | 2 each | 19.5 |
| Wednesday 2/14 | 2:30pm - 4pm, 5:30pm - 8pm(4 x3 = 12) | 2 each | 18 |
| Thursday 2/15  | Class (3:30-4:50), (5:00-6:00) (1.5 x3 = 4.5, 1x2 =2 ) | 2 each | 12.5 |
| Monday 2/19    |                             | 2 each | 6  |
| Tuesday 2/20   |                             | 4 each | 12 |
|                |                             |     | 87.5  |


*What aspects took the most time?*

    One aspect that we found really challenging was implementing both a button that can autofill the searched country, while simultaneously having buttons that would allow the user to select and deselect different features from our graph. When we got one to work, the other would break and for a while we got stuck spinning in circles trying different methods. In the end we discovered that our method for loading the legend was creating problems for the countries, because the legend was currently dependent on the colors of the features, which at the time were not preset, and when the colors would disappear because we were attempting to choose a new country, this would break our legend code causing an error. In the end we ended up preselecting colors for each feature, which decoupled the legend code from what was occurring on the graph which allowed both the buttons to work at the same time.
