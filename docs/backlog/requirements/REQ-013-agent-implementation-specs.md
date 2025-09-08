---
id: REQ-013
titel: Agent Implementation Specifications
type: technical
prioriteit: HOOG
status: backlog
aangemaakt: 08-09-2025
bijgewerkt: 08-09-2025
epic: EPIC-005
stories: []
bron: Technical design for agent enhancements
---

# REQ-013: Agent Implementation Specifications

## Beschrijving
Concrete technische implementatie specificaties voor het slimmer maken van Claude Code agents met backlog-awareness, inclusief exacte prompt updates, code patterns, en behavior modifications.

## Agent Prompt Enhancements

### 1. doc-standards-guardian Enhancement

#### Additional Prompt Instructions
```markdown
## BACKLOG AWARENESS CAPABILITIES

You now have backlog management capabilities. When reviewing documentation:

1. **Detect Backlog References**: 
   - Look for patterns: REQ-###, EPIC-###, US-###, BUG-### in code and docs
   - Validate these exist in: docs/requirements/, docs/epics/, docs/stories/
   - Report broken references

2. **Synchronize Documentation**:
   - When you see completed code for a story, check if:
     * README.md reflects the new functionality
     * CHANGELOG.md has an entry for the story
     * docs/INDEX.md is updated if needed
   
3. **Traceability Maintenance**:
   - When detecting new relationships, update TRACEABILITY-MATRIX.md:
   ```markdown
   | Requirement | Epic | Stories | Status |
   |------------|------|---------|--------|
   | REQ-007 | EPIC-004 | US-014, US-015 | in_progress |
   ```

4. **Auto-Update Behavior**:
   - If you find code comment like `// Implements US-014`, ensure:
     * The story file exists
     * Status is at least "in_progress"
     * Suggest updating to "review" if in PR

5. **Generate Missing Documentation**:
   ```python
   # If you see this in code:
   def dashboard_search():  # US-014
       pass
   
   # You should check/create:
   # docs/stories/US-014.md with proper metadata
   ```

IMPORTANT: Always ask before creating new backlog items, but proactively suggest when you detect missing documentation.
```

### 2. business-analyst-justice Enhancement

#### Additional Prompt Instructions
```markdown
## BACKLOG GENERATION CAPABILITIES

When translating requirements to development artifacts:

1. **Automatic Requirement Generation**:
   When user describes a need, create requirement with this template:
   ```yaml
   ---
   id: REQ-{next_number}
   titel: {extracted_title}
   type: functional|nonfunctional|domain|technical
   prioriteit: {analyze_urgency}
   status: backlog
   aangemaakt: {today}
   bijgewerkt: {today}
   epic: {suggest_epic}
   stories: []
   ---
   
   # Requirement {id}: {titel}
   
   ## Beschrijving
   {detailed_description}
   
   ## Acceptatiecriteria
   - [ ] {smart_criteria_1}
   - [ ] {smart_criteria_2}
   
   ## BDD Scenarios
   ```gherkin
   Gegeven {context}
   Wanneer {action}
   Dan {outcome}
   ```
   ```

2. **Epic Decomposition**:
   When creating epics, automatically decompose into stories:
   ```python
   epic_size = estimate_complexity(epic_description)
   if epic_size > 20:
       stories = decompose_epic(epic)
       for story in stories:
           create_user_story(story, epic_id)
   ```

3. **Story Point Estimation**:
   Use this rubric for automatic estimation:
   - 1-2 points: Single file change, no tests needed
   - 3-5 points: Multiple files, tests required
   - 8 points: New feature, multiple components
   - 13 points: Complex feature, architecture changes
   - 20+ points: Should be epic, not story

4. **Compliance Metadata Addition**:
   For justice sector projects, automatically add:
   ```yaml
   compliance:
     astra: [detect_relevant_controls]
     nora: [detect_relevant_principles]
     gemma: [detect_relevant_patterns]
   ```

5. **Smart Linking**:
   - Detect related requirements by keyword analysis
   - Suggest epic assignment based on domain
   - Identify dependencies from technical description
```

### 3. quality-assurance-tester Enhancement

#### Additional Prompt Instructions
```markdown
## TEST-BACKLOG INTEGRATION

When creating or maintaining tests:

1. **Test-Story Linking Pattern**:
   ```python
   # test_dashboard.py
   """
   @story: US-014
   @requirement: REQ-007
   @acceptance_criteria: [1, 2, 3]
   """
   
   def test_dashboard_search():
       """
       Acceptance Criterion 1: Search returns results in <100ms
       """
       # Test implementation
       pass
   ```

2. **Coverage Reporting**:
   After running tests, update story metadata:
   ```yaml
   # In US-014.md
   test_coverage:
     unit_tests: 85%
     integration_tests: 60%
     acceptance_criteria_covered: [1, 2, 3, 5]
     missing_coverage: [4]
   ```

3. **Automatic Bug Creation**:
   When test fails, create bug item:
   ```yaml
   ---
   id: BUG-{next_number}
   titel: {test_name} failure
   type: bug
   priority: {based_on_test_criticality}
   status: backlog
   story: {extracted_from_test_metadata}
   discovered_in: {test_file}:{line_number}
   ---
   
   ## Bug Description
   Test `{test_name}` failed with:
   ```
   {error_message}
   ```
   
   ## Steps to Reproduce
   1. Run `pytest {test_file}::{test_name}`
   
   ## Expected vs Actual
   - Expected: {expected_from_assertion}
   - Actual: {actual_result}
   ```

4. **Test Generation from Acceptance Criteria**:
   ```python
   def generate_tests_from_story(story_id):
       story = load_story(story_id)
       for criterion in story.acceptance_criteria:
           test_code = generate_test_skeleton(criterion)
           add_to_test_file(test_code, story_id)
   ```

5. **Definition of Done Validation**:
   Check and report:
   - [ ] All acceptance criteria have tests
   - [ ] Test coverage > 80%
   - [ ] All tests passing
   - [ ] Performance tests included if needed
```

### 4. workflow-router Enhancement

#### Additional Prompt Instructions
```markdown
## BACKLOG-AWARE ROUTING

Route work based on backlog item type and status:

1. **Item Type Detection**:
   ```python
   def detect_work_type(user_input):
       if "REQ-" in user_input:
           return route_to_requirement_workflow()
       elif "US-" in user_input:
           return route_to_story_workflow()
       elif "BUG-" in user_input:
           return route_to_bugfix_workflow()
       elif "EPIC-" in user_input:
           return route_to_epic_workflow()
   ```

2. **Status-Based Routing**:
   ```python
   item = load_backlog_item(item_id)
   
   workflows = {
       'backlog': planning_workflow,
       'in_progress': development_workflow,
       'review': review_workflow,
       'blocked': unblock_workflow,
       'done': documentation_workflow
   }
   
   return workflows[item.status]
   ```

3. **Automatic Status Transitions**:
   ```python
   # On workflow events:
   on_git_branch_create(story_id) -> update_status('in_progress')
   on_pr_create(story_id) -> update_status('review')
   on_pr_merge(story_id) -> update_status('done')
   on_test_fail(story_id) -> update_status('blocked')
   ```

4. **Escalation Rules**:
   ```yaml
   escalation:
     blocked_duration: 2_days -> notify_epic_owner
     no_progress: 3_days -> notify_team_lead
     overdue: 1_day -> increase_priority
   ```
```

### 5. developer-implementer Enhancement

#### Additional Prompt Instructions
```markdown
## STORY-DRIVEN DEVELOPMENT

When implementing code:

1. **Read Story Context**:
   ```python
   # Before starting implementation:
   story = load_story(story_id)
   display_acceptance_criteria(story)
   load_related_requirements(story)
   suggest_implementation_approach(story)
   ```

2. **Code Annotation Pattern**:
   ```python
   # @backlog: US-014
   # @requirement: REQ-007
   # @acceptance_criteria: 1
   def dashboard_search(query: str) -> List[Dict]:
       """
       Implements real-time search for dashboard (US-014).
       Fulfills requirement REQ-007 for HTML dashboard.
       """
       # Implementation
       pass
   ```

3. **Commit Message Enhancement**:
   ```bash
   # Automatic enhancement of commits:
   git commit -m "Add search" 
   # Becomes:
   git commit -m "feat(US-014): Add dashboard search functionality
   
   - Implements acceptance criteria 1 & 2
   - Real-time search with debouncing
   - Performance: <100ms response
   
   Story: US-014
   Requirement: REQ-007
   Epic: EPIC-004
   Remaining work: acceptance criteria 3-5"
   ```

4. **Time Tracking**:
   ```yaml
   # Auto-update in story file:
   estimated_hours: 8
   actual_hours: 6.5  # Auto-calculated from git time
   remaining_hours: 1.5
   ```

5. **PR Description Template**:
   ```markdown
   ## 📋 Story: US-014
   
   ### 🎯 Acceptance Criteria
   - [x] Criterion 1: Real-time search
   - [x] Criterion 2: Debouncing
   - [ ] Criterion 3: Highlighting
   
   ### 🧪 Testing
   - Unit tests: ✅
   - Integration tests: ✅
   - Coverage: 85%
   
   ### 📚 Documentation
   - README updated: ✅
   - API docs: ✅
   - CHANGELOG: ✅
   ```
```

### 6. code-reviewer-comprehensive Enhancement

#### Additional Prompt Instructions
```markdown
## BACKLOG-AWARE CODE REVIEW

When reviewing code:

1. **Acceptance Criteria Validation**:
   ```python
   def review_against_story(pr, story_id):
       story = load_story(story_id)
       checklist = []
       
       for criterion in story.acceptance_criteria:
           implemented = check_implementation(pr.changes, criterion)
           tested = check_test_coverage(pr.tests, criterion)
           documented = check_documentation(pr.docs, criterion)
           
           checklist.append({
               'criterion': criterion,
               'implemented': implemented,
               'tested': tested,
               'documented': documented
           })
       
       return generate_review_report(checklist)
   ```

2. **Definition of Done Checklist**:
   ```markdown
   ## Review Checklist for US-014
   
   ### Code Quality
   - [ ] No code smells
   - [ ] Follows project conventions
   - [ ] Proper error handling
   
   ### Story Completion
   - [ ] All acceptance criteria met
   - [ ] Tests coverage > 80%
   - [ ] Documentation updated
   
   ### Backlog Updates
   - [ ] Story status accurate
   - [ ] Hours tracked
   - [ ] Blockers documented
   ```

3. **Epic Progress Calculation**:
   ```python
   def update_epic_progress(story_id):
       story = load_story(story_id)
       epic = load_epic(story.epic)
       
       completed_points = sum(
           s.story_points for s in epic.stories 
           if s.status == 'done'
       )
       total_points = sum(s.story_points for s in epic.stories)
       
       epic.completion_percentage = (completed_points / total_points) * 100
       save_epic(epic)
   ```

4. **Review Comments Linking**:
   ```python
   # In review comment:
   "This doesn't meet acceptance criterion 3 of US-014"
   
   # Agent automatically:
   - Links comment to US-014
   - Marks criterion 3 as "needs work"
   - Suggests status change to "in_progress"
   ```
```

## Implementation Triggers

### Git Hooks Integration
```bash
#!/bin/bash
# .git/hooks/pre-commit

# Extract story ID from branch name
BRANCH=$(git branch --show-current)
STORY_ID=$(echo $BRANCH | grep -oE '(US|REQ|BUG)-[0-9]+')

if [ ! -z "$STORY_ID" ]; then
    # Update story status
    claude-code update-story $STORY_ID --status in_progress
    
    # Add story ID to commit message
    COMMIT_MSG=$(cat $1)
    echo "[$STORY_ID] $COMMIT_MSG" > $1
fi
```

### VSCode Extension Integration
```json
{
  "claude-code.backlog": {
    "enabled": true,
    "auto-detect-story": true,
    "show-story-status": true,
    "update-on-save": true
  }
}
```

### CLI Integration
```bash
# New claude-code commands
claude-code story start US-014
claude-code story status
claude-code story complete
claude-code sprint report
claude-code backlog sync
```

## Success Validation

### Agent Behavior Tests
```python
def test_agent_detects_story_in_code():
    code = "def search(): # US-014"
    agent_response = doc_standards_guardian.analyze(code)
    assert "US-014" in agent_response.detected_stories
    
def test_agent_updates_story_status():
    workflow_router.handle_pr_created("PR #123 for US-014")
    story = load_story("US-014")
    assert story.status == "review"
    
def test_agent_generates_test_from_criteria():
    story = load_story("US-014")
    tests = quality_assurance_tester.generate_tests(story)
    assert len(tests) == len(story.acceptance_criteria)
```

## Rollout Strategy

### Phase 1: Read-Only Mode
- Agents detect and report backlog items
- No automatic updates
- Gather metrics on accuracy

### Phase 2: Suggest Mode
- Agents suggest updates
- User confirms before applying
- Track acceptance rate

### Phase 3: Auto-Update Mode
- Automatic updates for safe operations
- Manual override always available
- Full audit trail

### Phase 4: Intelligent Mode
- Predictive suggestions
- Anomaly detection
- Self-improving accuracy