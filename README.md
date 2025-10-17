/// <summary>
/// Get active books by multiple subdesk IDs and date
/// </summary>
public async Task<List<BookSystemRegionMapActive>> GetActiveBooksBySubdesksAsync(
    List<int> subdeskIds,  // Changed from int to List<int>
    DateTime cobDate, 
    DateTime validDate)
{
    var query = from bsrm in _context.Set<BookSystemRegionMapActive>()
                join bh in _context.Set<BusinessHierarchy>() 
                    on bsrm.BookId equals bh.SrcBookId
                where bsrm.CobDate == cobDate
                    && subdeskIds.Contains(bh.SubdeskId.Value)  // IN clause
                    && validDate >= bh.ValidFrom
                    && validDate <= bh.ValidTo
                    && bh.GrainLevel == "Book"
                select bsrm;

    // Capture SQL query for debugging
    var sqlQuery = query.ToQueryString();

    return await query.Distinct().ToListAsync();
}


/// <summary>
/// Get active books by multiple subdesk IDs and date
/// </summary>
public async Task<List<BookSystemRegionMapActive>> GetActiveBooksBySubdesksAsync(
    List<int> subdeskIds,  // Changed from int to List<int>
    DateTime cobDate, 
    DateTime validDate)
{
    if (subdeskIds == null || subdeskIds.Count == 0)
        throw new ArgumentException("At least one subdesk ID is required", nameof(subdeskIds));

    return await _repository.GetActiveBooksBySubdesksAsync(subdeskIds, cobDate, validDate);
}


/// <summary>
/// Get active books by multiple subdesk IDs
/// </summary>
/// <param name="subdeskIds">Comma-separated subdesk IDs (e.g., 357909,551200,551350)</param>
/// <param name="cobDate">COB date</param>
/// <param name="validDate">Valid date for business hierarchy</param>
/// <returns>List of active books for the subdesks</returns>
[HttpGet("books")]
public async Task<ActionResult<List<BookSystemRegionMapActive>>> GetActiveBooksBySubdesks(
    [FromQuery] string subdeskIds,
    [FromQuery] DateTime cobDate, 
    [FromQuery] DateTime validDate)
{
    try
    {
        // Parse comma-separated string to list of integers
        var subdeskIdList = subdeskIds
            .Split(',')
            .Select(id => int.Parse(id.Trim()))
            .ToList();

        var books = await _service.GetActiveBooksBySubdesksAsync(subdeskIdList, cobDate, validDate);
        
        if (books == null || books.Count == 0)
            return NotFound($"No active books found for subdesks {subdeskIds}");
        
        return Ok(books);
    }
    catch (FormatException)
    {
        return BadRequest("Invalid subdesk IDs format. Use comma-separated integers.");
    }
    catch (ArgumentException ex)
    {
        return BadRequest(ex.Message);
    }
    catch (Exception ex)
    {
        return StatusCode(500, $"Internal server error: {ex.Message}");
    }
}
